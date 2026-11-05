---
title: "10. Validation and cleanup"
weight: 10
chapter: false
---

## End-to-end validation

Run the checks with the intended identities, not only as cluster admin. Save outputs with timestamps, but redact account IDs and endpoints before sharing.

### Network edge

- Public subnets have an internet-gateway route; private node subnets have no direct internet-gateway route.
- Nodes have no public IP addresses and the API endpoint is private or restricted to a documented admin CIDR.
- The ALB exposes only the intended HTTPS listener, uses an ACM certificate, and has the expected WAF association.
- An external client can reach the application hostname; a node public address is not an alternate entry path.

### Identity and workload

```bash
kubectl auth can-i create deployments -n payments
kubectl auth can-i get secrets -n payments
kubectl auth can-i get pods -n kube-system
kubectl get pods -n payments -o jsonpath='{range .items[*]}{.metadata.name}{"\t"}{.spec.serviceAccountName}{"\n"}{end}'
```

Run these as the DevOps, observer, and admin roles. Confirm expected allow/deny results. Confirm application pods use a dedicated service account and a workload role, not the node role. In a test namespace, verify restricted Pod Security Admission rejects the privileged sample.

### Network and detection

- Verify NetworkPolicy enforcement with a temporary client pod: allowed DNS and intended app traffic work; unrelated pod-to-pod traffic fails. Remove test pods afterward.
- Confirm EKS audit events arrive in CloudWatch and identify the actor for the ConfigMap test from Module 8.
- Confirm GuardDuty EKS coverage is active and test notifications reach an owned responder. A missing finding from a benign test is not proof the service is broken; validate coverage status and use AWS's documented test methods.
- Confirm incident responders know where logs and evidence are stored and who can revoke each access path.

## Remove lab resources

Record anything you want to retain, then remove resources in dependency order. Deleting a cluster does not necessarily delete its VPC, NAT gateways, load balancers, CloudWatch logs, IAM roles, S3 bucket, or GuardDuty configuration.

1. Remove the sample Ingress and wait for the ALB to be deleted:

   ```bash
   kubectl delete ingress web -n payments --ignore-not-found
   kubectl delete deployment web -n payments --ignore-not-found
   kubectl delete deployment blocked-client -n default --ignore-not-found
   kubectl delete namespace frontend --ignore-not-found
   kubectl delete networkpolicy --all -n payments
   kubectl delete namespace payments --ignore-not-found
   ```

2. In **EKS → Cluster → Compute**, delete the managed node group and wait until it is gone. Then delete the EKS cluster. Check the console for any remaining cluster-associated security groups or ENIs.
3. In **EC2 → Load Balancers** and **Target Groups**, confirm no lab ALB/target group remains. Delete unattached lab load balancers if the controller did not remove them.
4. Delete lab-only VPC Flow Logs, EventBridge rules, SNS topics/subscriptions, and any dedicated CloudTrail trail. Preserve or export audit data first. Do not remove organization trails or topics used by other workloads.
5. Delete the NAT gateway(s), then release their Elastic IPs after the NAT gateways finish deleting. Remove VPC endpoints created only for the lab.
6. Delete the lab VPC after its load balancers, NAT gateways, endpoints, ENIs, and other dependent resources are gone. The VPC console's delete flow lists blockers.
7. Remove only the IAM roles/policies and Identity Center assignments created for this lab (cluster role, node role, admin/devops/test access, VPC CNI role, Pod Identity workload role, and controller role). Do not delete shared organizational roles or permission sets.
8. Delete the lab CloudWatch log group only if you no longer need its evidence. Disable lab-only GuardDuty features only if they were not already enabled for the account and your security owner approves.
9. Empty and delete the dedicated sample S3 bucket and any dedicated CloudTrail log bucket only after preserving required records. Confirm each bucket name and contents before deletion.
10. If you created a customer-managed KMS key only for the cluster, confirm the cluster and all encrypted data are gone, then schedule key deletion according to your organization's key-retention policy. Never delete a shared key.

Keep audit evidence according to your retention requirement. Avoid broad `aws ... delete` commands copied from a lab unless every resource ID and account has been checked.

## Lessons and trade-offs

- A private node subnet still needs a deliberate path for image pulls, telemetry, and AWS APIs.
- An EKS access entry makes an IAM principal recognizable; the access policy or Kubernetes RBAC determines what it can do.
- Namespaces and NetworkPolicy improve isolation but do not equal separate kernels or separate accounts.
- GuardDuty and audit logs improve detection, but require an owner, a retention plan, and a tested response path.
- No single control guarantees safety. The useful result is several independently enforced boundaries plus a way to detect and recover when one fails.
