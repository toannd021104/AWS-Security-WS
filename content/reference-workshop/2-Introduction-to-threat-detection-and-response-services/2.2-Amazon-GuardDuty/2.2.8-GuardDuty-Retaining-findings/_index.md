---
title: "GuardDuty - Retaining findings"
date: "`r Sys.Date()`"
weight: 8
chapter: false
pre: " <b> 2.2.8 </b> "
---

## GuardDuty - Retaining findings

GuardDuty keeps findings for 90 days and can export them to Amazon EventBridge or S3 for long-term tracking. You can set how often findings are exported, and GuardDuty uses AWS KMS to encrypt data in your S3 bucket. Make sure your permissions are set up so GuardDuty can export findings securely.

Any new active findings that GuardDuty generates are automatically exported within about 5 minutes after the finding is generated. You can set the frequency for how often updates to the active findings are exported to EventBridge. The frequency that you select applies to the exporting of new occurrences of existing findings to EventBridge, your S3 bucket (when configured), and Detective (when integrated).

When you configure settings to export findings to an Amazon S3 bucket, GuardDuty uses AWS Key Management Service (AWS KMS) to encrypt the findings data in your S3 bucket. This requires you to add permissions to your S3 bucket and the AWS KMS key so that GuardDuty can use them to export findings in your account.
