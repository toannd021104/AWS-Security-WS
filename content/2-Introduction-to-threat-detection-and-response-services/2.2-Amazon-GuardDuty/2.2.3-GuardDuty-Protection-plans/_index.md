---
title: "GuardDuty - Protection plans"
date: "`r Sys.Date()`"
weight: 3
chapter: false
pre: " <b> 2.2.3 </b> "
---

## GuardDuty - Protection plans

#### Prerequisite

Before you start, make sure your EC2 instance has the SSM Management Role attached. If you need help, check the AWS documentation for adding instance profiles.

#### Amazon S3 protection in GuardDuty

1. Visit the S3 Protection page under Protection plans in GuardDuty.
2. Make sure S3 Protection is enabled to keep your storage safe.

#### EKS Protection in GuardDuty

3. Go to the EKS Protection page under Protection plans.
4. Ensure EKS Audit Log Monitoring is turned on. (Runtime Monitoring for EKS is now managed in a new feature tab.)

#### Runtime Monitoring in GuardDuty

5. Head to the Runtime Monitoring page under Protection plans.
6. Make sure Runtime Monitoring is enabled, including automated agent configuration for EKS, Fargate, and EC2.
7. Switch to the Runtime coverage tab to see coverage statistics. If you see issues like "No Agent Reporting," check your EC2 instance logs and AWS docs for troubleshooting tips.
   ![VPC](/images/2/2.2-Amazon-GuardDuty/2.2.3-GuardDuty-Protection-plans/s7a.png)

_Note_: If the EC2 Instance **Coverage status** is unhealthy and the **Issue** status is "**No Agent Reporting**" or something related to SSM
![VPC](/images/2/2.2-Amazon-GuardDuty/2.2.3-GuardDuty-Protection-plans/errorSSM.png)
it may be due to SSM agent can not be installed in the EC2 instance. You can clarify by check /var/log/amzn-guardduty-agent in the instance (AL2, AL2023). Further read https://docs.aws.amazon.com/guardduty/latest/ug/gdu-assess-coverage-ec2.html#ec2-runtime-monitoring-coverage-issues-troubleshoot

![VPC](/images/2/2.2-Amazon-GuardDuty/2.2.3-GuardDuty-Protection-plans/s7b.png)

#### Malware Protection for EC2 in Amazon GuardDuty

8. Visit the **Malware Protection** page under Protection plans in the GuardDuty console.

![VPC](/images/2/2.2-Amazon-GuardDuty/2.2.3-GuardDuty-Protection-plans/s8.png)

9. GuardDuty automatically initiates a malware scan after generating a finding indicative of malware in an EC2 instance or a container workload. Make sure that GuardDuty-initiated malware scan is enabled.
   ![VPC](/images/2/2.2-Amazon-GuardDuty/2.2.3-GuardDuty-Protection-plans/s9.png)
   Here are some EC2 malware scans
   ![VPC](/images/2/2.2-Amazon-GuardDuty/2.2.3-GuardDuty-Protection-plans/s9b.png)
   If you follow a scan through Scan ID
   ![VPC](/images/2/2.2-Amazon-GuardDuty/2.2.3-GuardDuty-Protection-plans/s9c.png)
   EICAR-Test file is a file to check whether the threat detection function works. It is not a real virus.

10. Scroll down and toggle **Retain scanned snapshots when malware is detected** on.

![VPC](/images/2/2.2-Amazon-GuardDuty/2.2.3-GuardDuty-Protection-plans/s10.png)

#### RDS Protection in Amazon GuardDuty

11. Visit the **RDS Protection** page under Protection plans in the GuardDuty console.

![VPC](/images/2/2.2-Amazon-GuardDuty/2.2.3-GuardDuty-Protection-plans/s11.png)

12. Make sure that RDS Login Activity Monitoring is enabled.

![VPC](/images/2/2.2-Amazon-GuardDuty/2.2.3-GuardDuty-Protection-plans/s12.png)

#### Lambda Protection in Amazon GuardDuty

13. Visit the **Lambda Protection** page under Protection plans in the GuardDuty console.

![VPC](/images/2/2.2-Amazon-GuardDuty/2.2.3-GuardDuty-Protection-plans/s13.png) 14. Make sure that Lambda Network Activity Monitoring is enabled.

![VPC](/images/2/2.2-Amazon-GuardDuty/2.2.3-GuardDuty-Protection-plans/s14.png)
