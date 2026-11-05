---
title: "5. Ingress and network policy"
weight: 5
chapter: false
---

## Goal

Allow the intended path from the public ALB to the app and deny unnecessary east-west traffic between pods. Use an AWS Load Balancer Controller for ingress resources; keep the Kubernetes API and worker nodes off the public internet.

## Important prerequisite: network policy enforcement

Kubernetes NetworkPolicy objects have no effect unless the network plugin enforces them. Amazon VPC CNI support depends on its version and network-policy feature configuration. Before applying policies, follow the current AWS VPC CNI documentation for your cluster version, enable the NetworkPolicy feature, and confirm the agent is healthy. A successful `kubectl apply` only proves the API stored the object; it does not prove packets are filtered.

Check the CNI pods and configuration:

```bash
kubectl -n kube-system get daemonset aws-node
kubectl -n kube-system get pods -l k8s-app=aws-node -o wide
```

Expected result: the `aws-node` DaemonSet has a ready pod on each Linux node. Consult the version-matched AWS guide before changing its ConfigMap or environment variables; enabling enforcement can change traffic behavior for every pod on a node.

For a managed add-on, enable it in **EKS → Cluster → Add-ons → Amazon VPC CNI → Edit**. Select a supported current add-on version (at least VPC CNI 1.21 for standard network policies), expand optional configuration, and set:

```json
{"enableNetworkPolicy":"true"}
```

Save the add-on configuration and wait for it to return to `ACTIVE`. The `aws-node` pods should have both the CNI and network-policy agent containers ready (`2/2`). EKS VPC CNI policy enforcement applies to Linux EC2 nodes; review AWS support limitations before mixing Fargate, Windows, alternate CNIs, or multiple pod network interfaces. In standard enforcement mode, newly started pods can briefly have default-allow behavior before their policy is programmed. Strict mode starts pods default-deny, but requires explicit policies for all required endpoints, including CoreDNS; use it only after testing system add-ons.

## Apply default-deny, then explicit allows

Start with a deny policy in the application namespace, then add narrowly scoped DNS and application rules. Test in a lab namespace first. The DNS exception below assumes CoreDNS pods have label `k8s-app=kube-dns` in `kube-system`; inspect your labels and adjust if they differ.

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: default-deny
  namespace: payments
spec:
  podSelector: {}
  policyTypes: [Ingress, Egress]
---
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-dns
  namespace: payments
spec:
  podSelector: {}
  policyTypes: [Egress]
  egress:
    - to:
        - namespaceSelector:
            matchLabels:
              kubernetes.io/metadata.name: kube-system
          podSelector:
            matchLabels:
              k8s-app: kube-dns
      ports:
        - protocol: UDP
          port: 53
        - protocol: TCP
          port: 53
```

Once DNS is confirmed, allow only the frontend path to the selected app port. The policy in `payments` selects source pods in a separate `frontend` namespace:

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-frontend-to-web
  namespace: payments
spec:
  podSelector:
    matchLabels:
      app: web
  policyTypes: [Ingress]
  ingress:
    - from:
        - namespaceSelector:
            matchLabels:
              kubernetes.io/metadata.name: frontend
          podSelector:
            matchLabels:
              app: frontend
      ports:
        - protocol: TCP
          port: 8080
```

Create the `frontend` namespace and a small non-root client Deployment so the allow rule has a real source to match:

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: frontend
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: frontend-client
  namespace: frontend
spec:
  replicas: 1
  selector:
    matchLabels:
      app: frontend
  template:
    metadata:
      labels:
        app: frontend
    spec:
      automountServiceAccountToken: false
      securityContext:
        runAsNonRoot: true
        runAsUser: 10001
        runAsGroup: 10001
        seccompProfile:
          type: RuntimeDefault
      containers:
        - name: client
          image: public.ecr.aws/docker/library/busybox:1.36
          command: ["sh", "-c", "sleep 3600"]
          securityContext:
            allowPrivilegeEscalation: false
            readOnlyRootFilesystem: true
            capabilities:
              drop: ["ALL"]
          resources:
            requests:
              cpu: 10m
              memory: 16Mi
            limits:
              cpu: 100m
              memory: 64Mi
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: blocked-client
  namespace: default
spec:
  replicas: 1
  selector:
    matchLabels:
      app: blocked-client
  template:
    metadata:
      labels:
        app: blocked-client
    spec:
      automountServiceAccountToken: false
      securityContext:
        runAsNonRoot: true
        runAsUser: 10001
        runAsGroup: 10001
        seccompProfile:
          type: RuntimeDefault
      containers:
        - name: client
          image: public.ecr.aws/docker/library/busybox:1.36
          command: ["sh", "-c", "sleep 3600"]
          securityContext:
            allowPrivilegeEscalation: false
            readOnlyRootFilesystem: true
            capabilities:
              drop: ["ALL"]
          resources:
            requests:
              cpu: 10m
              memory: 16Mi
            limits:
              cpu: 100m
              memory: 64Mi
```

Save as `frontend-client.yaml`, apply it with the policies, and wait for the client pod to become ready. Verify the allowed path and a denied path:

```bash
kubectl apply -f network-policies.yaml
kubectl apply -f frontend-client.yaml
kubectl rollout status deployment/frontend-client -n frontend
kubectl rollout status deployment/blocked-client -n default
kubectl exec -n frontend deploy/frontend-client -- \
  wget -T 3 -qO- http://web.payments.svc.cluster.local
kubectl exec -n default deploy/blocked-client -- \
  wget -T 3 -qO- http://web.payments.svc.cluster.local
```

The first request should return the NGINX welcome page. The second should time out or fail because `default` is not an allowed source namespace. If both requests work, first confirm `aws-node` has `2/2` containers and a `PolicyEndpoint` exists for the `web` pods before changing policies.

With the instance-target path used here, VPC CNI network policy allows traffic from a pod's own node in addition to the explicit policy rules. This is needed for node-level forwarding but means the NetworkPolicy is not the control that distinguishes ALB traffic from other traffic arriving through a node. The node security group and ALB security group provide that boundary; NetworkPolicy limits pod-to-pod and egress traffic. If you change to `target-type: ip`, re-design and test the ALB-to-pod ingress policy and security-group path instead of assuming the pod-selector example will match ALB traffic.

## Create an HTTPS ingress

Prerequisites: AWS Load Balancer Controller installed, its service account has its dedicated IAM role, public subnets are tagged for load balancer discovery, and an ACM certificate exists in the same region as the ALB. See AWS's versioned installation guide for controller IAM policy and Helm values; do not give the controller broad administrator permissions.

### Prepare the controller, certificate, and WAF

The AWS Load Balancer Controller watches Kubernetes Ingress resources and creates AWS load balancers. It is not the same component as the EKS control plane. The following command path pins the controller release (`v2.14.1`) and Helm chart (`1.14.0`); check AWS's installation page for newer supported releases when you repeat this workshop. The controller role is scoped to its own Kubernetes service account through this cluster's OIDC provider.

1. Associate the cluster's OIDC provider and download the release-matched IAM policy:

   ```bash
   export AWS_ACCOUNT_ID=$(aws sts get-caller-identity --query Account --output text)
   export LBC_VERSION=v2.14.1
   eksctl utils associate-iam-oidc-provider --cluster "$CLUSTER_NAME" \
     --region "$AWS_REGION" --approve
   curl -o iam_policy.json \
     "https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/${LBC_VERSION}/docs/install/iam_policy.json"
   aws iam create-policy --policy-name AWSLoadBalancerControllerIAMPolicy \
     --policy-document file://iam_policy.json
   ```

   Inspect `iam_policy.json` before creating the policy. If the policy name already exists, retrieve its ARN and review the current default version instead of creating a duplicate. In the console, this is **IAM → Policies → Create policy → JSON**.

2. Create the IAM role and `kube-system` service account association:

   ```bash
   eksctl create iamserviceaccount --cluster "$CLUSTER_NAME" \
     --namespace kube-system --name aws-load-balancer-controller \
     --attach-policy-arn "arn:aws:iam::${AWS_ACCOUNT_ID}:policy/AWSLoadBalancerControllerIAMPolicy" \
     --override-existing-serviceaccounts --region "$AWS_REGION" --approve
   ```

   This command uses CloudFormation to create an IAM role whose trust is limited to the controller service account. The console alternative is **IAM → Roles → Create role → Web identity**, select this cluster's OIDC provider, set audience `sts.amazonaws.com`, and subject `system:serviceaccount:kube-system:aws-load-balancer-controller`; attach only the release-matched controller policy.

3. Install the Gateway API standard CRDs required by the current AWS controller installation guide. This lab uses Ingress resources, not Gateway resources:

   ```bash
   kubectl apply --server-side -f \
     https://github.com/kubernetes-sigs/gateway-api/releases/download/v1.2.0/standard-install.yaml
   ```

4. Install the pinned Helm chart. Because this node group restricts IMDS, pass the region and VPC ID explicitly:

   ```bash
   helm repo add eks https://aws.github.io/eks-charts
   helm repo update eks
   helm install aws-load-balancer-controller eks/aws-load-balancer-controller \
     --namespace kube-system \
     --version 1.14.0 \
     --set clusterName="$CLUSTER_NAME" \
     --set region="$AWS_REGION" \
     --set vpcId="$VPC_ID" \
     --set serviceAccount.create=false \
     --set serviceAccount.name=aws-load-balancer-controller
   ```

5. Verify that the controller is available and its logs show no IAM or subnet-discovery errors:

   ```bash
   kubectl get deployment aws-load-balancer-controller -n kube-system
   kubectl logs -n kube-system deployment/aws-load-balancer-controller --tail=50
   ```

   Expected result: deployment is `2/2` available and logs do not show repeated `AccessDenied` or "unable to resolve at least one subnet" errors. Helm upgrades do not automatically update all CRDs; follow the AWS upgrade procedure when changing controller versions.

For the certificate, open **AWS Certificate Manager → Request certificate**, request a public certificate for a domain you control, choose DNS validation, and create the validation record in the authoritative hosted zone. Wait for status **Issued**. A certificate in another region cannot be attached to a regional ALB.

For WAF, open **AWS WAF & Shield → Web ACLs → Create web ACL**, choose **Regional** scope and the same region as the ALB, add a managed rule group appropriate for the app, and create the ACL. Start in count mode where possible, review false positives, then move to block mode. Record the Web ACL ARN. WAF has separate pricing and may block valid app traffic if rules are enabled without tuning.

The sample Service starts as `ClusterIP`. For this lab, use ALB **instance targets**: the ALB sends traffic to a NodePort on private worker nodes, and Kubernetes forwards it to the pod. Change the Service to `NodePort` before creating the Ingress:

```bash
kubectl patch service web -n payments -p '{"spec":{"type":"NodePort"}}'
kubectl get service web -n payments
```

Expected result: the Service shows a NodePort in the `30000-32767` range. Nodes have no public IP, and the controller should permit that node port only from the ALB security group. Verify the actual rules in EC2 after reconciliation.

Example Ingress (replace the certificate ARN and host):

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: web
  namespace: payments
  annotations:
    alb.ingress.kubernetes.io/scheme: internet-facing
    alb.ingress.kubernetes.io/target-type: instance
    alb.ingress.kubernetes.io/listen-ports: '[{"HTTPS":443}]'
    alb.ingress.kubernetes.io/certificate-arn: arn:aws:acm:us-east-1:123456789012:certificate/REPLACE_ME
    alb.ingress.kubernetes.io/ssl-redirect: "443"
    alb.ingress.kubernetes.io/wafv2-acl-arn: arn:aws:wafv2:us-east-1:123456789012:regional/webacl/REPLACE_ME
spec:
  ingressClassName: alb
  rules:
    - host: app.example.com
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: web
                port:
                  number: 80
```

Save the Ingress as `web-ingress.yaml`, then run `kubectl apply -f web-ingress.yaml` and `kubectl get ingress -n payments -w`. The ADDRESS should eventually show an ALB DNS name. Check **EC2 → Load Balancers → Security**: inbound should expose only the intended listener, and target access should be restricted to the ALB security group. Check **WAF & Shield → Web ACLs** to confirm the ACL is associated.

```bash
kubectl get service web -n payments
kubectl describe ingress web -n payments
kubectl get events -n payments --sort-by=.lastTimestamp | tail -20
```

Expected result: the Service has ready endpoints, the Ingress has an ALB address, and its events do not show reconciliation errors. If the Ingress remains pending, check the controller logs, public subnet tags, IAM policy/role association, and security group rules before opening additional ports.

Expected outcome: HTTPS requests to the configured hostname reach the app; HTTP is redirected or unavailable; direct access to node addresses is not part of the design. ALB terminates TLS and sends HTTP to the NodePort inside the VPC in this example. If policy requires encryption to the target, configure HTTPS target protocol and certificates on the app as well. A newly created ALB can take several minutes. `ADDRESS` remaining blank usually means controller/IAM/subnet discovery errors; inspect controller logs before changing network rules.

## Console inspection and screenshot

**Screenshot to capture:** the ALB listener showing HTTPS and its ACM certificate, the attached WAF web ACL, and target health. Redact public DNS names if the endpoint should not be shared. Also capture the network policy YAML and a successful/denied connectivity test from test pods.

## Trade-offs and common failures

- A public ALB is intentional for a public app; a private ALB is better for internal services. Pick the scheme explicitly.
- WAF protects HTTP(S) requests at the ALB/CloudFront layer; it does not filter pod-to-pod traffic.
- Default-deny egress can break DNS, telemetry, image pulls from in-pod tools, and application dependencies. Add only required destinations and ports, then observe logs and test.
- NetworkPolicy semantics for host-network pods and traffic to node-local services vary. Verify using the chosen CNI and version.
- `target-type: ip` is convenient and avoids NodePort traffic, but requires correct subnet discovery, security-group behavior, and pod routability.

### Further reading

- [AWS Load Balancer Controller installation](https://docs.aws.amazon.com/eks/latest/userguide/lbc-helm.html)
- [Route application traffic with an ALB](https://docs.aws.amazon.com/eks/latest/userguide/alb-ingress.html)
- [Enable VPC CNI network policy](https://docs.aws.amazon.com/eks/latest/userguide/cni-network-policy-configure.html)
- [AWS WAF web ACLs](https://docs.aws.amazon.com/waf/latest/developerguide/web-acl.html)
