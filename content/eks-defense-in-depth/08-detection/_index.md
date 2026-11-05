---
title: "8. Audit logs and threat detection"
weight: 8
chapter: false
---

## Goal

Collect enough evidence to answer: who called the Kubernetes API, which AWS identity was used, what changed, and whether a workload showed suspicious behavior. Logging is useful only when retention, access, and response ownership are defined.

## Confirm control-plane logs

In **EKS → Cluster → Logging → Manage logging**, enable `api`, `audit`, `authenticator`, `controllerManager`, and `scheduler`. Choose a retention period in CloudWatch Logs that matches your incident and compliance requirements; indefinite retention can grow cost. Restrict who can read and delete the log group, and consider exporting long-term evidence to a separate security account with retention controls.

Make a harmless API request and locate it:

```bash
kubectl get namespaces
aws logs describe-log-groups --region "$AWS_REGION" \
  --log-group-name-prefix "/aws/eks/$CLUSTER_NAME/cluster"
```

Expected result: the cluster log group exists and an audit stream appears after the API request. Audit events include the user identity and verb; inspect the event's `user.username`, `verb`, `objectRef`, and `sourceIPs` fields. Avoid logging or sharing full request bodies that may contain sensitive data.

Set the log group's retention in **CloudWatch → Log groups → select group → Actions → Edit retention setting**. Choose a duration that matches your investigation window, then confirm a cluster operator cannot delete the logs. If policy requires stronger separation, deliver a copy to a security account or protected S3 bucket.

## Enable AWS-side identity and network evidence

- **CloudTrail:** in **CloudTrail → Trails**, inspect whether an organization trail already covers this account. If not, create a multi-region trail for management events, deliver to a protected S3 bucket, enable log-file validation, and restrict delete access. EKS access-entry changes, IAM changes, and many network-control changes are AWS API events. Event History is useful for recent management events; a trail supports longer retention and central review and may incur storage costs.
- **VPC Flow Logs:** in **VPC → Your VPCs → select the lab VPC → Flow logs → Create flow log**, select all traffic and send to a dedicated CloudWatch log group (or a protected S3 destination). Use an approved delivery role or let the console create one. Flow Logs record network metadata and accept/reject status, not packet contents. Set retention and restrict log access.
- **GuardDuty:** open **GuardDuty → EKS Protection** and enable Kubernetes audit log monitoring for the region; then open **Runtime Monitoring → EKS Runtime Monitoring** and enable it for the lab account/cluster where supported. Runtime coverage requires the supported GuardDuty agent/configuration and compatible nodes. Review region support, prerequisites, exclusions, and pricing before enabling it. If GuardDuty was already enabled by the account security team, do not change organization-wide settings for this lab.
- **Amazon Inspector:** use it for supported compute and container image vulnerability findings; it does not replace runtime threat detection or admission policy.

**Screenshot to capture:** GuardDuty protection plan showing EKS Protection and the runtime coverage state, with account identifiers and findings redacted. Record which region and cluster are in scope.

## Make a test finding without attacking the cluster

Use a safe administrative change as an audit test: as the lab administrator, create a ConfigMap in the `payments` namespace, then inspect the audit event:

```bash
kubectl create configmap audit-check -n payments --from-literal=purpose=lab
kubectl get configmap audit-check -n payments
```

In CloudWatch Logs Insights, select the EKS audit log group and run a basic query (field names can vary with EKS audit event shape):

```text
fields @timestamp, user.username, verb, objectRef.namespace, objectRef.resource, objectRef.name
| filter objectRef.name = "audit-check"
| sort @timestamp desc
| limit 20
```

Expected result: a `create` event for `configmaps/audit-check` identifies the caller and namespace. If fields are not extracted automatically, use `parse @message` or inspect the raw JSON and adapt the query. Delete the test object after review.

## Alert on useful signals

Start with actionable events: repeated denied API requests, unexpected access-entry/role changes, creation of privileged workloads, GuardDuty high-severity EKS findings, and changes to public endpoint CIDRs. For a first notification path, create an SNS topic and confirm an email subscription, then create an **Amazon EventBridge → Rules → Create rule** rule on the default event bus with this GuardDuty pattern:

```json
{
  "source": ["aws.guardduty"],
  "detail-type": ["GuardDuty Finding"],
  "detail": {
    "severity": [{"numeric": [">=", 7]}]
  }
}
```

Set the target to the SNS topic and let EventBridge create its invoke role. Generate a harmless test through the service's supported sample-finding action, confirm delivery, and remove the test finding. Route alerts to an owned incident channel in production and include cluster, region, finding ID, timestamp, and runbook link. Avoid sending raw secrets or full pod environments into chat notifications.

Test one alert end to end. Confirm detection, notification, on-call ownership, and a documented response. A dashboard with no owner or no response path is not an operational control.

## Retention and access trade-offs

More detailed logs improve investigation but increase ingestion, storage, and query cost. Restrict log-group deletion and security-tool administration separately from cluster administration. Set retention based on the time needed to discover and investigate incidents; export evidence to a protected destination if it must outlive the CloudWatch retention window. Confirm that workload logs do not contain credentials before broadening access to them.

### Further reading

- [Control plane logs for Amazon EKS](https://docs.aws.amazon.com/eks/latest/userguide/control-plane-logs.html)
- [GuardDuty EKS Protection](https://docs.aws.amazon.com/guardduty/latest/ug/kubernetes-protection.html)
- [GuardDuty Runtime Monitoring](https://docs.aws.amazon.com/guardduty/latest/ug/runtime-monitoring.html)
- [CloudWatch Logs Insights query syntax](https://docs.aws.amazon.com/AmazonCloudWatch/latest/logs/CWL_QuerySyntax.html)
- [Create a multi-region CloudTrail trail](https://docs.aws.amazon.com/awscloudtrail/latest/userguide/cloudtrail-create-and-update-a-trail.html)
- [VPC Flow Logs](https://docs.aws.amazon.com/vpc/latest/userguide/flow-logs.html)
