---
title: "Detective - Overview"
date: "`r Sys.Date()`"
weight: 1
chapter: false
pre: " <b> 2.4.1 </b> "
---

## Detective - Overview

Amazon Detective is your AWS investigation assistant! It collects log data and uses smart analysis to help you quickly find the root cause of security issues. Detective builds a linked set of data so you can easily explore and investigate suspicious activities. It keeps up to a year of data and provides visualizations to show changes in activity and link them to security findings.

Amazon Detective simplifies the investigative process and helps security teams conduct faster and more effective investigations. Amazon Detective’s prebuilt data aggregations, summaries, and context help you to quickly analyze and determine the nature and extent of possible security issues. Amazon Detective maintains up to a year of aggregated data and makes it easily available through a set of visualizations that shows changes in the type and volume of activity over a selected time window, and links those changes to security findings. There are no upfront costs and you pay only for the events analyzed, with no additional software to deploy or log feeds to enable.

Amazon Detective extracts time-based events such as login attempts, API calls, and network traffic from AWS CloudTrail, Amazon Virtual Private Cloud (Amazon VPC) Flow Logs, Amazon GuardDuty findings, AWS Security Hub findings, and Amazon Elastic Kubernetes Service (Amazon EKS) audit logs. Detective creates a behavior graph that utilizes machine learning (ML) to create a unified, interactive view of your resource behaviors and their interactions over time, specifically for these time-based events. By exploring the behavior graph, you can analyze security events such as failed login attempts, suspicious APIs call, or finding groups to help you in investigating the root cause of your AWS Security Findings.

Threat actors often perform a series of actions when attempting to compromise your AWS environment, which can result in multiple security findings across your AWS resources. Finding groups are collections of security findings and resources that are associated with a single potential security incident you should investigate together. Finding groups can help reduce triage time because you don’t have to investigate each individual security finding separately. You can start your investigation with finding groups, which offer a more complete understanding of the incident. It also offers interactive visualizations that allows you to explore specific findings and insights using generative AI to describe the chain of events in natural language.
