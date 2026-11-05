---
title: "EKS Defense in Depth Workshop"
date: 2026-11-05
weight: 1
chapter: false
---

# Defend Amazon EKS in Depth

This workshop takes a request from the public internet to a Kubernetes pod and asks, at every boundary: **who is allowed through, what can they reach, and how will we know if that control fails?**

It is written for readers who know the basics of AWS but are new to EKS security. You will build a layered baseline on a real EKS cluster, use the AWS Console for inspection and selected setup steps, and use AWS CLI, `kubectl`, Helm, and Kubernetes YAML where they make the configuration repeatable.

{{% notice warning %}}
This lab creates billable AWS resources. Use a non-production account, record the resource names you create, and complete the cleanup module. Costs vary by region and usage; NAT gateways, EKS control planes, EC2 nodes, load balancers, log ingestion, and GuardDuty features can all incur charges.
{{% /notice %}}

## What you will build

An internet-facing application enters through a load balancer in public subnets. EKS worker nodes and pods run in private subnets. The cluster API is private or restricted to a known administration network. Human access uses individual IAM roles and EKS access entries; Kubernetes RBAC scopes namespace permissions. Workloads receive only the AWS permissions they need. Pod security, network policy, audit logging, and runtime detection add independent controls around the workload.

## Workshop modules

1. [Architecture, threat model, and preparation](eks-defense-in-depth/01-architecture/)
2. [Internet edge, VPC, and subnet boundaries](eks-defense-in-depth/02-network/)
3. [EKS control plane and cluster access](eks-defense-in-depth/03-cluster/)
4. [Node, namespace, and pod hardening](eks-defense-in-depth/05-workloads/)
5. [Ingress and network policy](eks-defense-in-depth/06-traffic/)
6. [IAM, EKS access entries, and Kubernetes RBAC](eks-defense-in-depth/04-identity/)
7. [Workload identity and secrets](eks-defense-in-depth/07-workload-identity/)
8. [Audit logs and threat detection](eks-defense-in-depth/08-detection/)
9. [Incident containment and recovery](eks-defense-in-depth/09-response/)
10. [Validation and cleanup](eks-defense-in-depth/10-cleanup/)

The [original AWS security workshop](reference-workshop/) remains available as reference material while this EKS workshop is developed.

## How to use the labs

Each module explains the security goal, the choice being made, console navigation where useful, implementation steps, expected output, and verification. Console screenshot prompts are marked **Screenshot to capture**. Capture your own account's screens; account IDs, endpoints, and findings are environment-specific.
