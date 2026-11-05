---
title: "2. Internet edge, VPC, and subnets"
weight: 2
chapter: false
---

## Goal

Put internet-facing load balancers and private EKS nodes in different subnets, then make the routes and security groups reflect that boundary. A subnet is not private because of its name; it is private because its route table has no direct route to an internet gateway.

## Create the VPC in the console

1. Open **VPC → Your VPCs → Create VPC**. Choose **VPC and more**.
2. Name it `eks-depth-lab`, choose an IPv4 CIDR such as `10.20.0.0/16`, and select two Availability Zones.
3. Create two public and two private subnets. Use non-overlapping ranges, for example public `10.20.0.0/20`, `10.20.16.0/20`; private `10.20.32.0/20`, `10.20.48.0/20`.
4. For a simple lab, create one NAT gateway per AZ for private-subnet outbound access. This is resilient but can be a significant fixed hourly cost. For a short-lived learning cluster, one NAT gateway costs less but creates an AZ dependency. VPC endpoints can reduce NAT use, but each endpoint has its own cost and must cover the services your nodes and pods call.
5. Keep **Enable DNS hostnames** and **Enable DNS resolution** enabled. Create the VPC and note its ID, subnet IDs, route tables, and NAT gateway ID.

**Screenshot to capture:** VPC resource map showing public subnets routed to the internet gateway and private subnets routed through NAT. Confirm the diagram is for the lab VPC before capturing it.

## Verify the route boundary

In **VPC → Subnets**, select each subnet and inspect its associated route table. Public subnets should have a default route to an internet gateway (`igw-...`). Private subnets should have a default route to a NAT gateway (`nat-...`) or an intentionally designed set of VPC endpoints, and no direct internet-gateway route.

```bash
export VPC_ID=vpc-REPLACE_ME
aws ec2 describe-route-tables --region "$AWS_REGION" \
  --filters "Name=vpc-id,Values=$VPC_ID" \
  --query 'RouteTables[].{RouteTable:RouteTableId,Associations:Associations[].SubnetId,Routes:Routes[].{Destination:DestinationCidrBlock,Gateway:GatewayId,Nat:NatGatewayId}}' \
  --output table
```

Expected result: the public route tables show `0.0.0.0/0` to `igw-...`; private route tables show `0.0.0.0/0` to `nat-...` when using NAT. A route table without an explicit subnet association may be the VPC's main table, so inspect the associations rather than relying on its name.

## Tag subnets for EKS and the AWS Load Balancer Controller

EKS components use subnet tags to discover where to place load balancers. Add these tags in **VPC → Subnets → Tags**. Replace IDs with the matching subnets:

| Subnet type | Tag key | Value |
|---|---|---|
| Public, for internet-facing load balancers | `kubernetes.io/role/elb` | `1` |
| Private, for internal load balancers | `kubernetes.io/role/internal-elb` | `1` |
| All cluster subnets | `kubernetes.io/cluster/eks-depth-lab` | `shared` |

The cluster tag helps controllers discover ownership. Do not tag a subnet public merely to make discovery work: its actual route table determines exposure.

## Security group intent

The EKS control plane, nodes, and ALB use different security groups. Keep inbound rules narrow:

- Internet to ALB: TCP 443 from the intended client CIDRs (or `0.0.0.0/0` only when the application is deliberately public). Redirect HTTP to HTTPS if port 80 is enabled.
- ALB to worker nodes: only the service's assigned NodePort from the ALB security group. The controller normally manages the target rule for instance targets; verify it after the ALB is created. Never expose the NodePort range to the internet.
- Operator to Kubernetes API: TCP 443 from the VPC or approved administration network, not the whole internet.
- Node-to-node and node-to-control-plane traffic: use EKS-created rules and the documented ports needed by the selected CNI and cluster components. Avoid replacing generated rules without understanding their dependencies.

Security groups are stateful. A return packet for an allowed connection is permitted automatically, but that does not make a broad inbound rule safe. Network ACLs are stateless and are not a substitute for a clear security-group policy.

## Cost and availability trade-off

Two NAT gateways improve AZ independence but add fixed cost. A single NAT gateway is cheaper for a disposable lab, but if its AZ fails, private nodes in the other AZ lose that egress path. Interface endpoints can keep AWS API traffic private, but the endpoint list must match the image-pull, logging, identity, and application requirements. Document the choice instead of assuming "private subnet" means no internet dependency.

### Checkpoint

Before continuing, identify the public subnet IDs, private subnet IDs, route tables, NAT choice, and VPC ID. Nodes will use only private subnets; the load balancer will use public subnets for public traffic.

### Further reading

- [EKS VPC and subnet requirements](https://docs.aws.amazon.com/eks/latest/userguide/network-reqs.html)
- [Subnet tagging requirements for load balancers](https://docs.aws.amazon.com/eks/latest/userguide/alb-ingress.html)
