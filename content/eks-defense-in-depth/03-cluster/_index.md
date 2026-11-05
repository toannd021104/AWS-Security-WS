---
title: "3. EKS control plane and cluster access"
weight: 3
chapter: false
---

## Goal

Create an EKS control plane with private worker nodes, an API endpoint reachable only from trusted networks, and audit logs that can later answer who changed cluster state. The EKS control plane is AWS-managed; your responsibility is to configure its access paths and retain useful evidence.

## Create the cluster in the console

1. Open **Amazon EKS → Clusters → Add cluster → Create**.
2. Set the name to `eks-depth-lab`. Select a Kubernetes version currently marked supported in your region. Keep the EKS cluster IAM role created for the cluster service; if the wizard has no suitable role, choose **Create recommended role**. This service role is not a human administrator role.
3. On **Networking**, select the VPC and the **private subnets** from Module 2 for the cluster's control-plane ENIs. Choose a cluster security group and leave required EKS communication rules intact.
4. Set the Kubernetes API endpoint to **Private** for the intended design. If the console flow requires temporary public access to bootstrap from your workstation, select **Public and private**, set public access to a specific trusted CIDR, finish bootstrap from that network, then disable public access. Never leave unrestricted public API access as a convenience.
5. In **Cluster access**, select **EKS API** and leave **Bootstrap cluster administrator access** enabled for this lab's initial setup. The cluster creator is temporarily the only Kubernetes administrator; use that identity only to install the baseline and create the named access entries in Module 6. You will test the separate admin role before removing the bootstrap path. In a production pipeline, pre-create and verify the intended access entry before disabling bootstrap admin access.
6. Enable control-plane log types **api**, **audit**, **authenticator**, **controllerManager**, and **scheduler**. Logs are delivered to CloudWatch Logs and incur ingestion/storage charges.
7. In **Add-ons**, choose the managed **Amazon VPC CNI**, **CoreDNS**, and **kube-proxy** add-ons if the wizard offers them. The later NetworkPolicy and workload-identity labs expect the managed VPC CNI add-on. If your console created these as self-managed components, migrate them using the AWS add-on procedure before continuing.
8. In **Secrets encryption**, enable envelope encryption for Kubernetes Secrets if available and select an approved KMS key. Record the key; deleting or disabling it can make encrypted data unavailable. If your lab account has no approved customer-managed key, document the choice and configure it before using real secrets.
9. Review the summary and create the cluster. Wait for status **Active** before continuing.

The cluster API endpoint being private means `kubectl` must run on a host that can route to the VPC and resolve the private endpoint. If your laptop is outside the VPC, use an approved VPN, Direct Connect, or an in-VPC administration host. Do not broaden the endpoint just to make a local command work.

{{% notice note %}}
With a private-only EKS endpoint, `aws eks update-kubeconfig` can run outside the VPC, but Kubernetes API requests (`kubectl`) cannot. Run `kubectl` from an approved connected host. EKS console operations use AWS APIs and do not require your browser to resolve the private Kubernetes endpoint.
{{% /notice %}}

**Screenshot to capture:** cluster networking page showing the selected VPC, private subnets, API endpoint setting, and authentication mode. Redact account-specific IDs if publishing the image.

## Verify control-plane configuration

```bash
aws eks describe-cluster --name "$CLUSTER_NAME" --region "$AWS_REGION" \
  --query 'cluster.{Status:status,Version:version,Endpoint:resourcesVpcConfig.endpoint,Public:resourcesVpcConfig.endpointPublicAccess,Private:resourcesVpcConfig.endpointPrivateAccess,PublicCidrs:resourcesVpcConfig.publicAccessCidrs,Auth:accessConfig.authenticationMode,Logs:logging.clusterLogging[?enabled==`true`].types}' \
  --output yaml
```

Expected result: `Status` is `ACTIVE`, `Private` is `true`, `Public` is `false` for the final private-only setup, `Auth` includes `API`, and the enabled log types include the five selected above. A public endpoint can coexist with a private endpoint, but if used temporarily its CIDR list should be narrow and the temporary path should be removed.

## Add a managed node group

1. In **IAM → Roles → Create role**, choose **AWS service → EC2**, then attach `AmazonEKSWorkerNodePolicy` and `AmazonEC2ContainerRegistryPullOnly`. Name it `EKSDepthNodeRole`. Do not reuse the cluster service role or a human administrator role. Prefer assigning `AmazonEKS_CNI_Policy` to a dedicated role for the VPC CNI add-on rather than leaving pod-IP management permissions on every node role.
2. Open the cluster's **Compute** tab and choose **Add node group**. Select the node role you created.
3. Choose private subnets only. Start with two nodes across two AZs for a resilient learning lab; select an instance family that is available and suitable for the sample app. Set desired and minimum size to `2`; keep maximum at `2` for this cost-controlled lab unless you are testing autoscaling.
4. Do not assign public IP addresses. Use the private-subnet NAT/endpoints from Module 2 for image pulls and required AWS APIs. If you use Pod Identity with no NAT, create the `com.amazonaws.<region>.eks-auth` interface VPC endpoint with private DNS so the agent can call EKS Auth.
5. Keep remote SSH access disabled. Prefer Systems Manager Session Manager for break-glass host access only when the node and IAM setup explicitly support it.
6. Create the node group and wait for status **Active**.

To block ordinary pods from reaching the node's EC2 instance profile, create an EC2 launch template before creating the node group: open **EC2 → Launch templates → Create launch template → Advanced details → Metadata options**, require IMDSv2, and set the response hop limit to `1`. Select that launch template when creating the managed node group. Do not add subnet IDs, a network interface, or an instance profile to the template; EKS manages those settings for a managed node group. A hop limit of 1 can break workloads that rely on IMDS, so test the cluster add-ons and use EKS Pod Identity/IRSA for AWS credentials. If you already created nodes without a hardened template, roll out a replacement node group and move workloads gradually rather than changing every node at once.

**Screenshot to capture:** launch-template metadata options showing IMDSv2 required and hop limit `1`, with launch-template IDs redacted.

## Connect from an approved network

```bash
aws eks update-kubeconfig --name "$CLUSTER_NAME" --region "$AWS_REGION"
kubectl cluster-info
kubectl get nodes -o wide
```

Expected result: `cluster-info` reports the Kubernetes API and `get nodes` shows the managed nodes as `Ready`. If the request times out, first check DNS and routing from your admin host to the private endpoint; an IAM policy cannot fix missing network reachability.

## Review audit logs

In **CloudWatch → Log groups**, open `/aws/eks/eks-depth-lab/cluster`. The `audit` stream records Kubernetes API requests; the `authenticator` stream helps investigate identity mapping. New log streams may take several minutes to appear after the first events. An empty log group immediately after cluster creation does not prove logging is disabled; generate an API request and check again.

### Checkpoint

The control plane is active, worker nodes are private, the endpoint is reachable only from the intended admin network, and control-plane logs are enabled. Next, separate AWS identity from Kubernetes authorization so each operator gets only the actions they need.

### Further reading

- [Create an Amazon EKS cluster](https://docs.aws.amazon.com/eks/latest/userguide/create-cluster.html)
- [Configure the EKS API server endpoint](https://docs.aws.amazon.com/eks/latest/userguide/config-cluster-endpoint.html)
- [Create a managed node group](https://docs.aws.amazon.com/eks/latest/userguide/create-managed-node-group.html)
