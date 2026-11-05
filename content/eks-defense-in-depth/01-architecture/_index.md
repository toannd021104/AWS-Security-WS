---
title: "1. Architecture and threat model"
weight: 1
chapter: false
---

## Goal

Before clicking through the console, decide which paths should exist. Defense in depth means a failure at one boundary does not automatically grant access to the next layer. A private subnet helps, but it does not replace IAM, RBAC, pod security, or monitoring.

## Reference architecture

```text
Internet users
    | HTTPS :443
    v
Route 53 -> (optional CloudFront/WAF) -> internet-facing ALB
                                      public subnets, 2+ AZs
                                                |
                                                | target traffic :NodePort
                                                v
                                   private worker subnets
                                   EKS nodes / VPC CNI
                                                |
                                   ingress -> Service -> pods
                                                |
                                NetworkPolicy + namespace boundary

Operators -> IAM Identity Center / IAM role -> EKS access entry -> RBAC
Workloads -> Kubernetes service account -> Pod Identity / IRSA -> AWS API
EKS control plane -> CloudTrail + control-plane logs + GuardDuty
```

The ALB is the only internet-facing application entry point in this design. Nodes have no public IP addresses. The Kubernetes API endpoint is private for normal operations; if you enable public access during a temporary bootstrap, restrict it to a known administration CIDR and remove that path afterward. A private endpoint requires the operator to have network connectivity to the VPC, such as VPN, Direct Connect, a bastion, or an in-VPC runner.

## Threats and controls

| Threat | Primary control | Independent backstop |
|---|---|---|
| Internet scans node or API addresses | Private nodes; restricted API endpoint | Security groups, audit logs, GuardDuty |
| Compromised app calls AWS APIs | Pod Identity/IRSA with least privilege | Short-lived credentials, CloudTrail, role scoping |
| Developer gets cluster-admin accidentally | Individual access entries and namespace RBAC | `kubectl auth can-i`, CloudTrail review |
| Vulnerable pod reaches another workload | Default-deny NetworkPolicy and scoped allow rules | Separate namespaces, runtime detection |
| Malicious or misconfigured container | Restricted Pod Security Admission | Admission review, image scanning, runtime alerts |
| Credentials or secrets leak | No static AWS keys; external secret store where needed | Rotation, access audit, incident procedure |

## Decide the lab values

Use a unique short prefix and record the values. The examples below assume `eks-depth-lab`, region `us-east-1`, and two Availability Zones; change them consistently if your region differs.

```bash
export AWS_REGION=us-east-1
export CLUSTER_NAME=eks-depth-lab
export LAB_PREFIX=eks-depth-lab
aws sts get-caller-identity
aws eks list-clusters --region "$AWS_REGION"
```

Expected result: `get-caller-identity` returns the AWS account and principal you intend to use. If it returns a production identity or an unexpected role, stop and switch profiles before creating resources. Keep this identity separate from the role you will later use as a cluster administrator.

## Lab boundaries

- Use a non-production account and do not put real customer data or credentials in the sample workload.
- Create resources in one region. EKS clusters, VPCs, load balancers, and many security services are regional.
- Plan for two public and two private subnets across two AZs. EKS and load balancer availability depend on subnet/AZ placement.
- Save the cluster name, VPC ID, subnet IDs, IAM role ARNs, and created resource names. The final module uses them for cleanup.
- Review every console confirmation page before choosing **Create**. This lab intentionally enables logging and security features that may add usage charges.

{{% notice note %}}
**Screenshot to capture:** one architecture diagram annotated with public/private subnet boundaries and the two identity paths (human role to RBAC, workload service account to AWS role). Avoid recording account IDs or credential values in screenshots.
{{% /notice %}}

### Further reading

- [Amazon EKS best practices: security](https://docs.aws.amazon.com/eks/latest/best-practices/security.html)
- [Amazon EKS best practices: network security](https://docs.aws.amazon.com/eks/latest/best-practices/network-security.html)
