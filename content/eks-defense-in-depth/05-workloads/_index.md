---
title: "4. Node, namespace, and pod hardening"
weight: 4
chapter: false
---

## Goal

Reduce what a compromised container can do to the node and cluster. A container is not a virtual machine: it shares the host kernel. Restrict privileges, use supported node images, and enforce a namespace-level baseline so a bad manifest is rejected before it runs.

## Harden the managed node group

In **EKS → Cluster → Compute → Node groups**, confirm the group uses private subnets and has no SSH remote access configured. Prefer EKS-optimized Amazon Linux images and managed node group updates. Schedule updates: a managed node group does not remove the need to patch Kubernetes components and host packages.

For workloads that need AWS APIs, do not let pods inherit the node instance role. In **EC2 → Instances → select a node → Security**, confirm the attached instance profile is the dedicated node role. Review its permissions and remove application-specific policies from it. Workload permissions are handled in Module 7.

EKS-managed node groups use an instance profile and EC2 metadata service. Use the supported EKS/AMI configuration for IMDSv2 and prevent ordinary pods from reaching instance metadata. The launch-template step in Module 3 requires IMDSv2 and sets the response hop limit to `1`, which commonly prevents a packet traversing a pod network namespace from reaching IMDS. Verify this against your CNI and host-network workloads before enforcing it; a host-network pod can have different reachability. Do not apply an untested launch-template change to every node at once.

## Enforce the Pod Security Standards baseline

Kubernetes Pod Security Admission is built in. Label an application namespace to enforce the **restricted** standard, while using `warn` and `audit` first to discover incompatible workloads. Do this in a test namespace before production:

```bash
kubectl create namespace payments
kubectl label namespace payments \
  pod-security.kubernetes.io/audit=restricted \
  pod-security.kubernetes.io/warn=restricted \
  pod-security.kubernetes.io/enforce=restricted \
  pod-security.kubernetes.io/enforce-version=latest --overwrite
```

`latest` follows the cluster's current policy version; pin a Kubernetes minor version in a production change process when predictable policy behavior matters. Enforcement can reject old charts, sidecars, and debugging pods. Review warnings and audit events before enabling `enforce` on an existing namespace.

## Deploy a restricted sample pod

Create namespace `payments` as above. Apply this Deployment:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web
  namespace: payments
spec:
  replicas: 2
  selector:
    matchLabels:
      app: web
  template:
    metadata:
      labels:
        app: web
    spec:
      automountServiceAccountToken: false
      securityContext:
        runAsNonRoot: true
        seccompProfile:
          type: RuntimeDefault
      containers:
        - name: web
          image: nginxinc/nginx-unprivileged:stable-alpine
          ports:
            - name: http
              containerPort: 8080
          securityContext:
            allowPrivilegeEscalation: false
            readOnlyRootFilesystem: true
            capabilities:
              drop: ["ALL"]
          resources:
            requests:
              cpu: 50m
              memory: 64Mi
            limits:
              cpu: 250m
              memory: 128Mi
          volumeMounts:
            - name: tmp
              mountPath: /tmp
      volumes:
        - name: tmp
          emptyDir: {}
```

Save it as `web-deployment.yaml`, then:

```bash
kubectl apply -f web-deployment.yaml
kubectl rollout status deployment/web -n payments
kubectl get pods -n payments
```

Expected result: two pods reach `Running` and `READY 1/1`. The unprivileged NGINX image listens on port 8080 and supplies its own non-root UID, so the pod can drop all Linux capabilities without trying to bind privileged port 80. The tag is mutable; for a reproducible deployment, resolve and pin an approved digest, and scan it before promotion.

Expose the app only inside the cluster first. Add this Service to `web-deployment.yaml` (or save it as `web-service.yaml`):

```yaml
apiVersion: v1
kind: Service
metadata:
  name: web
  namespace: payments
spec:
  type: ClusterIP
  selector:
    app: web
  ports:
    - name: http
      port: 80
      targetPort: 8080
```

Apply it and verify the selector finds both pods:

```bash
kubectl apply -f web-service.yaml
kubectl get service,endpointslice -n payments
```

Expected result: the Service has a ClusterIP and EndpointSlices list the ready pod IPs. `ClusterIP` keeps the service internal; Module 5 will add the controlled ALB entry point.

## Prove the admission control rejects an unsafe pod

Try a deliberately noncompliant pod in the test namespace:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: must-be-rejected
  namespace: payments
spec:
  containers:
    - name: shell
      image: public.ecr.aws/docker/library/busybox:1.36
      command: ["sh", "-c", "sleep 3600"]
      securityContext:
        privileged: true
```

Apply the file with `kubectl apply -f unsafe-pod.yaml`. Expected result: the API rejects it with a Pod Security violation (privileged container is not allowed by `restricted`). Delete the local sample file if you do not need it. This is a safe policy test; it does not exploit a running workload.

## Namespace boundaries are not complete isolation

Namespaces scope names and some RBAC policies. They do not isolate the kernel, node, or network by themselves. Combine them with NetworkPolicy, workload identity, admission controls, and separate node groups or clusters when a workload's risk or compliance boundary requires stronger isolation. Avoid giving application service accounts permission to create Pods, RoleBindings, or privileged workloads; those permissions can become an indirect privilege escalation path.

### Further reading

- [Pod Security Standards](https://kubernetes.io/docs/concepts/security/pod-security-standards/)
- [EKS security best practices for Pods](https://docs.aws.amazon.com/eks/latest/best-practices/pod-security.html)
- [EKS identity and access management best practices](https://docs.aws.amazon.com/eks/latest/best-practices/identity-and-access-management.html)
