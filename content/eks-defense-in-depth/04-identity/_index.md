---
title: "6. Human access, IAM, and Kubernetes RBAC"
weight: 6
chapter: false
---

## Authentication and authorization happen in layers

`kubectl` access is a chain, not a single permission:

1. **AWS IAM authenticates the caller** and allows EKS token generation and cluster discovery.
2. **An EKS access entry** associates the IAM principal with EKS access. It can grant an EKS-managed access policy or map the principal to Kubernetes groups.
3. **Kubernetes authorization** decides whether the authenticated identity may get, create, update, or delete a specific resource. This is where namespace-scoped RBAC can constrain normal users and DevOps roles.

IAM `eks:DescribeCluster` alone does not grant Kubernetes API permissions. Conversely, a Kubernetes RoleBinding does not make an IAM principal able to authenticate. Keep these layers distinct while troubleshooting.

## Create separate operator roles

Use IAM Identity Center permission sets or assumable IAM roles for individual people. Avoid shared IAM users and long-lived access keys. Create separate roles for at least:

- `EKSDepthAdmin`: break-glass cluster administrators; tightly assigned and audited.
- `EKSDepthDevOps`: deployment operators scoped to the application namespace.
- `EKSDepthReadOnly`: support or audit users who can inspect approved resources.

In **IAM Identity Center → Users**, create or select named users, then assign each user to the AWS account with a purpose-specific permission set. Use a custom permission set policy that grants only `eks:DescribeCluster` on this cluster ARN for kubeconfig generation; the EKS access entry below supplies Kubernetes authorization. Do not put Kubernetes admin permissions into a broad AWS administrator policy. For users federated through another identity provider, create equivalent short-lived assumable roles and require MFA through the identity provider.

Example AWS permission policy for these operators (replace account and region):

```json
{
  "Version": "2012-10-17",
  "Statement": [{
    "Effect": "Allow",
    "Action": ["eks:DescribeCluster"],
    "Resource": "arn:aws:eks:us-east-1:123456789012:cluster/eks-depth-lab"
  }]
}
```

This AWS policy lets the CLI discover the cluster endpoint and certificate; it does not grant Kubernetes permissions. If you use the AWS Console to inspect Kubernetes resources, add only the console-specific EKS read actions required for that workflow and retain the same EKS authorization scope.

**Screenshot to capture:** IAM Identity Center account assignments showing distinct permission sets for admin, DevOps, and read-only access. Redact user names and account IDs before publishing.

Each role needs the AWS-side ability to call `eks:DescribeCluster` and obtain an EKS token through the supported authentication flow. Attach only the specific IAM permissions required for these operations; do not attach `AdministratorAccess` as a shortcut. The IAM policy is not a substitute for the EKS/Kubernetes permissions below.

Set the role ARNs from your account (do not paste an STS assumed-role session ARN). Example:

```bash
export ADMIN_ROLE_ARN=arn:aws:iam::123456789012:role/EKSDepthAdmin
export DEVOPS_ROLE_ARN=arn:aws:iam::123456789012:role/EKSDepthDevOps
export READONLY_ROLE_ARN=arn:aws:iam::123456789012:role/EKSDepthReadOnly
```

## Add an administrator entry

From the account bootstrap identity, create the access entry and associate the cluster-admin policy:

```bash
aws eks create-access-entry --cluster-name "$CLUSTER_NAME" \
  --principal-arn "$ADMIN_ROLE_ARN" --type STANDARD --region "$AWS_REGION"

aws eks associate-access-policy --cluster-name "$CLUSTER_NAME" \
  --principal-arn "$ADMIN_ROLE_ARN" \
  --policy-arn arn:aws:eks::aws:cluster-access-policy/AmazonEKSClusterAdminPolicy \
  --access-scope type=cluster --region "$AWS_REGION"
```

Expected result: both operations return an access-entry or association response. The cluster-admin policy is powerful; use this principal only for controlled administration and recovery, not daily application deployment.

## Give DevOps access to one namespace

The `payments` namespace is created in Module 4. Confirm its name before attaching a namespace-scoped access policy:

```bash
kubectl get namespace payments
```

Use an EKS-managed policy scoped to the application namespace for ordinary deployments:

```bash
aws eks create-access-entry --cluster-name "$CLUSTER_NAME" \
  --principal-arn "$DEVOPS_ROLE_ARN" --type STANDARD --region "$AWS_REGION"

aws eks associate-access-policy --cluster-name "$CLUSTER_NAME" \
  --principal-arn "$DEVOPS_ROLE_ARN" \
  --policy-arn arn:aws:eks::aws:cluster-access-policy/AmazonEKSEditPolicy \
  --access-scope type=namespace,namespaces=payments --region "$AWS_REGION"
```

Verify the policy's documented permissions. `AmazonEKSEditPolicy` is more permissive than read-only and may allow creating workloads that consume namespace resources; resource quotas and pod security still matter. The bootstrap operator creating these entries needs the corresponding EKS management API permissions.

Treat permission to create or update workloads as access to the identities those workloads can use. A DevOps user who can create Pods in a namespace may be able to set `serviceAccountName` to an existing service account and receive that workload's AWS role. Keep high-impact service accounts in a namespace where only trusted operators can deploy, and grant each workload role only the specific AWS actions and resources it needs.

For a support user, use `AmazonEKSViewPolicy` scoped to `payments` instead of edit. It provides view access, not access to Secret contents. If a user needs a narrower custom permission set, use an access entry with a Kubernetes group and a hand-written Role/RoleBinding rather than granting cluster-wide access.

## Custom RBAC for application support

Example: map a human support role to a group that can inspect pods and logs in `payments` but cannot create workloads or read Secrets. The group name is an authorization label, not a username or AWS role ARN.

```bash
aws eks create-access-entry --cluster-name "$CLUSTER_NAME" \
  --principal-arn "$READONLY_ROLE_ARN" --type STANDARD \
  --kubernetes-groups payments-observers --region "$AWS_REGION"
```

Apply this Role and RoleBinding as a cluster administrator:

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: payments-observer
  namespace: payments
rules:
  - apiGroups: [""]
    resources: ["pods", "services", "events"]
    verbs: ["get", "list", "watch"]
  - apiGroups: [""]
    resources: ["pods/log"]
    verbs: ["get"]
---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: payments-observers
  namespace: payments
subjects:
  - kind: Group
    name: payments-observers
    apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: Role
  name: payments-observer
  apiGroup: rbac.authorization.k8s.io
```

Save as `payments-observer-rbac.yaml` and run `kubectl apply -f payments-observer-rbac.yaml` as an administrator. The namespace was created earlier. Do not add `secrets`, `serviceaccounts/token`, `pods/exec`, `nodes`, or wildcard resources unless the job requires them and the risk is accepted.

## Test each identity

Assume each role through your normal identity provider, update kubeconfig, and test its effective permissions:

```bash
aws eks update-kubeconfig --name "$CLUSTER_NAME" --region "$AWS_REGION"
kubectl auth can-i get pods -n payments
kubectl auth can-i create deployments -n payments
kubectl auth can-i get secrets -n payments
kubectl auth can-i get pods -n kube-system
```

Expected results:

- Admin: `yes` for cluster resources (use only for controlled admin work).
- DevOps: `yes` for permitted application namespace operations; `no` outside its scope.
- Observer: `yes` for pods/logs in `payments`, `no` for creating deployments, reading Secrets, or accessing other namespaces.

The exact managed-policy permissions can evolve; the `can-i` checks are the operational proof for the cluster you built. A successful `kubectl get` is not enough to prove least privilege. Test both allowed and denied actions.

## Retire the temporary bootstrap administrator

The cluster-creation identity has admin access so the earlier modules can be completed. After the named admin role works, inspect access entries in **EKS → Cluster → Access → Access entries** or with:

```bash
aws eks list-access-entries --cluster-name "$CLUSTER_NAME" --region "$AWS_REGION"
```

Identify the original creator principal by its exact IAM role ARN. Confirm the new `EKSDepthAdmin` role can run `kubectl get nodes` and `kubectl auth can-i '*' '*'` from an approved network. Then remove or disassociate only the creator's cluster-admin entry in the console. Keep the bootstrap AWS identity's `eks:DescribeCluster` permission if it still needs to run EKS management commands; that AWS permission alone does not grant Kubernetes access.

**Screenshot to capture:** access-entry list with distinct admin, DevOps, and observer principals and their scopes. Redact account IDs and role paths before sharing.

{{% notice warning %}}
Do not remove the only administrative access entry until a second tested admin path exists. Keep the bootstrap role protected and avoid using the root account for routine access.
{{% /notice %}}

### Further reading

- [EKS access entries](https://docs.aws.amazon.com/eks/latest/userguide/access-entries.html)
- [EKS access policies](https://docs.aws.amazon.com/eks/latest/userguide/access-policies.html)
- [Kubernetes RBAC](https://kubernetes.io/docs/reference/access-authn-authz/rbac/)
