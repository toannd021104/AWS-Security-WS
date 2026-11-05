---
title: "7. Workload identity and secrets"
weight: 7
chapter: false
---

## Goal

Give an application a short-lived AWS identity without storing access keys in its image, environment, or Kubernetes Secret. EKS Pod Identity is the path used here; IRSA is a valid alternative when its OIDC trust model better fits your organization.

## Move VPC CNI permissions off the node role

The VPC CNI needs EC2 permissions to allocate pod IP addresses. AWS recommends a separate IAM role for its `kube-system/aws-node` service account. This is a cluster add-on identity, distinct from both the node role and application roles. The procedure below uses IRSA; the same CNI add-on also supports EKS Pod Identity.

The AWS Load Balancer Controller module created the cluster's IAM OIDC provider. Confirm that one exists, then create a role using **IAM → Roles → Create role → Web identity** or `eksctl`:

```bash
eksctl create iamserviceaccount --cluster "$CLUSTER_NAME" \
  --namespace kube-system --name aws-node \
  --role-name AmazonEKSVPCCNIRole \
  --attach-policy-arn arn:aws:iam::aws:policy/AmazonEKS_CNI_Policy \
  --override-existing-serviceaccounts --region "$AWS_REGION" --approve
```

Expected result: the `aws-node` service account has an IAM role annotation scoped to this cluster's OIDC provider and the `kube-system:aws-node` subject. Existing pods do not receive changed service-account annotations, so roll the DaemonSet and watch it recover one node at a time:

```bash
kubectl rollout restart daemonset/aws-node -n kube-system
kubectl rollout status daemonset/aws-node -n kube-system --timeout=10m
kubectl get pods -n kube-system -l k8s-app=aws-node -o wide
```

If networking degrades, restore the previous node-role permission before continuing.

After confirming the CNI role works, remove `AmazonEKS_CNI_Policy` from the node role in IAM if it was attached during node-role setup. Do not remove the policy before the add-on has assumed its replacement role. This workshop's recommended node role omits it; the check matters if the console created a broader role.

## Why the node role is not the app role

If every pod can use the EC2 node role, one compromised container may inherit permissions intended for the whole node group. Give each workload a separate IAM role and grant only the API actions and resource ARNs it needs. Keep the node role for node operations such as joining the cluster and pulling images.

## Create the application service account

Create a dedicated service account; do not let the workload use the namespace's default identity:

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: web-reader
  namespace: payments
automountServiceAccountToken: false
```

For EKS Pod Identity, EKS injects a separate projected, short-lived token and credential-provider settings into pods that use an associated service account. The application does not need a Kubernetes API token, so `automountServiceAccountToken: false` can remain. Never bind this service account to a Kubernetes Role merely to obtain AWS credentials.

```bash
kubectl apply -f web-reader-serviceaccount.yaml
aws eks create-addon --cluster-name "$CLUSTER_NAME" \
  --addon-name eks-pod-identity-agent --region "$AWS_REGION"
```

If the add-on already exists, inspect its status instead of creating it again:

```bash
aws eks describe-addon --cluster-name "$CLUSTER_NAME" \
  --addon-name eks-pod-identity-agent --region "$AWS_REGION" \
  --query 'addon.{Status:status,Version:addonVersion}' --output table
```

Expected result: add-on status `ACTIVE`. The cluster needs the Pod Identity Agent on nodes and a compatible AWS SDK in the application.

If private nodes have no NAT route, create the interface endpoint **com.amazonaws.<region>.eks-auth** in the VPC, enable private DNS, and allow TCP 443 from the node security group. This endpoint is required for the agent to call EKS Auth; the pod-to-agent link-local address alone is not enough.

## Create a least-privilege role

This example allows `GetObject` only under one S3 prefix. Create a bucket for the lab and replace the account, region, bucket, and cluster values before using the policy. If the bucket uses a customer-managed KMS key, the role also needs the narrowly scoped KMS decrypt permission and key policy permission.

In **Amazon S3 → Create bucket**, use a globally unique name, keep **Block all public access** enabled, and leave default encryption enabled. Upload a harmless file such as `payments/example.txt` and another file under `unrelated/` for the allow/deny check. Record the bucket name; do not use a production bucket.

Trust policy `pod-identity-trust.json`:

```json
{
  "Version": "2012-10-17",
  "Statement": [{
    "Effect": "Allow",
    "Principal": {"Service": "pods.eks.amazonaws.com"},
    "Action": ["sts:AssumeRole", "sts:TagSession"],
    "Condition": {
      "StringEquals": {
        "aws:RequestTag/eks-cluster-arn": "arn:aws:eks:us-east-1:123456789012:cluster/eks-depth-lab",
        "aws:RequestTag/kubernetes-namespace": "payments",
        "aws:RequestTag/kubernetes-service-account": "web-reader"
      }
    }
  }]
}
```

Create the role and the permissions policy. These IAM changes can also be made in **IAM → Roles**; the CLI is shown so the policy document is reviewable and repeatable. The trust conditions bind the role to this cluster, namespace, and service account. For accounts in AWS Organizations, consider also requiring `aws:SourceOrgId` as an additional confused-deputy guard.

```bash
aws iam create-role --role-name EKSDepthPaymentsWebReader \
  --assume-role-policy-document file://pod-identity-trust.json
```

Save as `payments-s3-read.json`:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": ["s3:GetObject"],
      "Resource": "arn:aws:s3:::REPLACE_WITH_BUCKET/payments/*"
    },
    {
      "Effect": "Allow",
      "Action": ["s3:ListBucket"],
      "Resource": "arn:aws:s3:::REPLACE_WITH_BUCKET",
      "Condition": {"StringLike": {"s3:prefix": ["payments", "payments/*"]}}
    }
  ]
}
```

```bash
aws iam put-role-policy --role-name EKSDepthPaymentsWebReader \
  --policy-name ReadPaymentsObjects --policy-document file://payments-s3-read.json
```

Check the trust policy is limited to the intended cluster and account. Avoid `s3:*`, `Resource: "*"`, and a trust policy that lets arbitrary clusters assume the role.

## Associate the IAM role with the service account

```bash
export WORKLOAD_ROLE_ARN=arn:aws:iam::123456789012:role/EKSDepthPaymentsWebReader
aws eks create-pod-identity-association --cluster-name "$CLUSTER_NAME" \
  --namespace payments --service-account web-reader \
  --role-arn "$WORKLOAD_ROLE_ARN" --region "$AWS_REGION"
```

Expected result: the response includes an association ID. Associations are scoped to a cluster, namespace, and service account. Confirm it:

```bash
aws eks list-pod-identity-associations --cluster-name "$CLUSTER_NAME" \
  --namespace payments --region "$AWS_REGION" --output table
```

The same association can be created in **EKS → Cluster → Access → Pod Identity → Create** by selecting the IAM role, namespace `payments`, and service account `web-reader`. The identity creating the association needs `iam:PassRole` for this workload role and permission to create EKS Pod Identity associations.

Use `serviceAccountName: web-reader` in the application pod template. Roll out the workload after the association exists so the Pod Identity Agent can provide credentials.

## Verify from inside a test workload

Use a temporary AWS CLI Job with `serviceAccountName: web-reader`; do not print credentials. Save this as `identity-check.yaml`:

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: identity-check
  namespace: payments
spec:
  template:
    spec:
      restartPolicy: Never
      serviceAccountName: web-reader
      automountServiceAccountToken: false
      securityContext:
        runAsNonRoot: true
        runAsUser: 10001
        runAsGroup: 10001
        seccompProfile:
          type: RuntimeDefault
      containers:
        - name: awscli
          image: public.ecr.aws/aws-cli/aws-cli:2
          command: ["sh", "-c", "sleep 3600"]
          securityContext:
            allowPrivilegeEscalation: false
            readOnlyRootFilesystem: true
            capabilities:
              drop: ["ALL"]
          env:
            - name: HOME
              value: /tmp
          volumeMounts:
            - name: tmp
              mountPath: /tmp
      volumes:
        - name: tmp
          emptyDir: {}
```

Apply it, then run:

```bash
kubectl apply -f identity-check.yaml
kubectl wait -n payments --for=condition=Ready pod \
  -l job-name=identity-check --timeout=120s
export IDENTITY_POD=$(kubectl get pods -n payments -l job-name=identity-check \
  -o jsonpath='{.items[0].metadata.name}')
kubectl exec -n payments "$IDENTITY_POD" -- aws sts get-caller-identity
kubectl exec -n payments "$IDENTITY_POD" -- \
  aws s3api list-objects-v2 --bucket REPLACE_WITH_BUCKET --prefix payments/
kubectl exec -n payments "$IDENTITY_POD" -- \
  aws s3api list-objects-v2 --bucket REPLACE_WITH_BUCKET --prefix unrelated/
```

Expected: the first command returns the workload role ARN; the permitted prefix works; the unrelated prefix is denied. A failed STS request often means the Pod Identity Agent, trust conditions, service-account association, or node egress is wrong. Do not fix it by attaching the policy to the node role.

## Store application secrets separately

Use AWS Secrets Manager or Systems Manager Parameter Store for application secrets. Retrieve them through the workload role or a supported Secrets Store CSI/External Secrets integration, scoped to specific secret ARNs. Enable customer-managed KMS keys only when key ownership or separation requirements justify the added key-policy operations. Never put credentials in a ConfigMap, image layer, Git repository, or screenshot.

EKS Kubernetes Secret data is base64-encoded, not encrypted by base64. Review EKS envelope encryption for Kubernetes API secrets, restrict `get/list/watch` on `secrets`, and avoid exposing secret values through logs or environment dumps. Secret rotation is not complete until the workload reloads the new value and old credentials are revoked.

### Further reading

- [EKS Pod Identity](https://docs.aws.amazon.com/eks/latest/userguide/pod-identities.html)
- [Configure the Pod Identity Agent](https://docs.aws.amazon.com/eks/latest/userguide/pod-id-agent-setup.html)
- [Assign an IAM role to a Kubernetes service account](https://docs.aws.amazon.com/eks/latest/userguide/pod-id-association.html)
- [Configure the VPC CNI to use IRSA](https://docs.aws.amazon.com/eks/latest/userguide/cni-iam-role.html)
