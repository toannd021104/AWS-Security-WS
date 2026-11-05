# EKS Defense in Depth Workshop

A practical, English-language workshop for building a layered security baseline around Amazon EKS. It follows traffic and identity from the internet edge through a VPC, cluster, nodes, namespaces, and pods, then shows how to detect and respond to suspicious activity.

## Start here

Open the [workshop landing page](content/_index.md), then follow the modules under [`content/eks-defense-in-depth`](content/eks-defense-in-depth). The original AWS Threat Detection & Response workshop is retained under [`content/reference-workshop`](content/reference-workshop) as a reference for the existing Hugo Learn structure and lab style.

## Lab assumptions

- You have an AWS account where you are allowed to create EKS, VPC, IAM, EC2, load balancer, logging, and GuardDuty resources.
- The hands-on path uses a real EKS cluster. AWS charges for the EKS control plane and for supporting resources such as NAT gateways, EC2 nodes, load balancers, log ingestion, and GuardDuty features.
- Use a non-production account. The workshop includes access-control and isolation changes that should be reviewed before applying to a production cluster.
- You have AWS CLI v2, `kubectl`, `helm`, `eksctl`, and `curl` installed. Console steps are included where they help learners inspect AWS settings or capture screenshots.

Each module states prerequisites, implementation steps, expected results, and verification commands. Clean up the lab resources when finished.
