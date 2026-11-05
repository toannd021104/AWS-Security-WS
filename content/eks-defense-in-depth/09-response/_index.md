---
title: "9. Incident containment and recovery"
weight: 9
chapter: false
---

## Scenario

GuardDuty or an operator reports unexpected behavior from a pod in `payments`. The goal is to limit impact while retaining enough evidence to understand what happened. Do not start by deleting the pod: that can remove useful process state and logs, and a Deployment may immediately recreate it.

## 1. Open an incident record and identify scope

Record the finding ID, UTC time, region, cluster, namespace, pod, node, service account, image digest, and suspected AWS role. Use a separate trusted admin identity. Do not paste tokens, Secret values, or customer data into the ticket.

```bash
kubectl get pods -n payments -o wide
kubectl describe pod -n payments POD_NAME
kubectl get pod -n payments POD_NAME -o jsonpath='{.spec.serviceAccountName}{"\n"}{.spec.nodeName}{"\n"}{.status.containerStatuses[*].imageID}{"\n"}'
kubectl get events -n payments --sort-by=.lastTimestamp
```

Expected result: the pod's service account, node, image digest, restarts, and recent events are recorded. Save the relevant audit, application, and GuardDuty records to the approved evidence store before retention windows expire.

## 2. Stop new work and preserve evidence

Pause the deployment pipeline or GitOps reconciliation for the affected workload so it does not recreate or overwrite the state during containment. Save deployment/ReplicaSet/pod YAML, current image digest, logs, and relevant audit events. Use your approved forensic procedure before taking node-level actions; do not run ad hoc tools inside a suspected compromised container.

For a rapidly spreading event, containment can take priority over complete evidence collection. Record what was changed and when.

## 3. Restrict workload traffic

Apply a quarantine policy to the affected pod labels or namespace. This example denies ingress and egress for pods labeled `incident=quarantine`; first confirm the CNI enforces NetworkPolicy. The empty ingress and egress lists mean no traffic is allowed by this policy selector, except behavior allowed by the CNI/platform itself.

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: incident-quarantine
  namespace: payments
spec:
  podSelector:
    matchLabels:
      incident: quarantine
  policyTypes: [Ingress, Egress]
```

Apply the policy and label only the affected pod:

```bash
kubectl apply -f incident-quarantine.yaml
kubectl label pod -n payments POD_NAME incident=quarantine --overwrite
kubectl get networkpolicy -n payments incident-quarantine
```

Expected result: policy exists and the pod has the quarantine label. NetworkPolicy is not a universal emergency kill switch: enforcement can be asynchronous, host-network traffic differs, and some CNI paths may not be covered. If immediate network isolation is required, use the documented AWS security-group or node-isolation procedure for your architecture and verify the traffic is actually blocked.

## 4. Revoke access and contain AWS impact

- If a human IAM role is compromised, remove or disable the affected identity assignment/session through your identity provider, review CloudTrail, and rotate exposed credentials. Remove the EKS access entry if Kubernetes access must stop immediately.
- If a workload role may be abused, remove the Pod Identity association and narrow or deny the role's permissions. Review CloudTrail for the role session and affected resources. Already issued STS credentials may remain usable until expiry unless an effective explicit deny or revocation control blocks them; verify impact in the target service rather than assuming association removal instantly invalidates every session.
- If the node may be compromised, cordon it to prevent new scheduling, preserve evidence using the approved procedure, then replace the node from a trusted managed node group/AMI. Do not treat deleting one pod as remediation for a compromised host.

```bash
kubectl cordon NODE_NAME
aws eks list-pod-identity-associations --cluster-name "$CLUSTER_NAME" \
  --namespace payments --region "$AWS_REGION"
```

Only cordon a node after assessing workload availability. Cordon prevents new scheduling but does not evict or isolate existing pods.

## 5. Eradicate, restore, and learn

Rebuild the workload from a reviewed image digest and trusted source. Fix the entry point (vulnerable dependency, leaked secret, permissive RBAC, or network path) before restoring traffic. Rotate application secrets and verify old credentials are rejected. Restore normal labels and policies only after the owner approves recovery.

Close the incident with a timeline, affected identities/resources, evidence locations, containment actions, recovery checks, and a concrete prevention change. Update the detection rule and runbook, then run a tabletop exercise to check the procedure without creating a real compromise.

{{% notice warning %}}
Containment commands can interrupt service or destroy forensic evidence. Follow the organization's incident authority and evidence-retention policy. This lab scenario is a defensive exercise; do not simulate credential theft or malware in an account containing production resources.
{{% /notice %}}

### Further reading

- [AWS security incident response guide](https://docs.aws.amazon.com/whitepapers/latest/aws-security-incident-response-guide/welcome.html)
- [EKS security best practices](https://docs.aws.amazon.com/eks/latest/best-practices/security.html)
