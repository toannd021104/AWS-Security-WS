---
title: "GuardDuty - Notifications"
date: "`r Sys.Date()`"
weight: 7
chapter: false
pre: " <b> 2.2.7 </b> "
---

## GuardDuty - Notifications

#### Configure SNS topic

1. Go to Amazon SNS in the AWS console.
2. Click Topics in the left navigation.
3. Click Create topic.
4. Choose Standard type.
5. Name your topic "guardduty-findings".
6. Leave other settings as default and click Create topic.

#### Subscribe to the topic

7. On the topic page, click Create subscription.
8. For Protocol, select email.
9. Enter your email address to receive notifications. You can unsubscribe after the workshop.
10. Click Create subscription.
11. Check your email for a confirmation message and click the confirmation link.

#### Create an EventBridge Rule to send findings to the topic

12. Navigate to Amazon EventBridge in the AWS console.
13. Click Create rule.
14. Name your rule "guardduty-findings".
15. In the Event pattern section, choose Custom pattern.
16. Enter the following pattern to filter for GuardDuty findings with CRITICAL severity:

```json
{
  "source": ["aws.guardduty"],
  "detail-type": ["GuardDuty Finding"],
  "detail": {
    "severity": [
      4, 4.0, 4.1, 4.2, 4.3, 4.4, 4.5, 4.6, 4.7, 4.8, 4.9, 5, 5.0, 5.1, 5.2,
      5.3, 5.4, 5.5, 5.6, 5.7, 5.8, 5.9, 6, 6.0, 6.1, 6.2, 6.3, 6.4, 6.5, 6.6,
      6.7, 6.8, 6.9, 7, 7.0, 7.1, 7.2, 7.3, 7.4, 7.5, 7.6, 7.7, 7.8, 7.9, 8,
      8.0, 8.1, 8.2, 8.3, 8.4, 8.5, 8.6, 8.7, 8.8, 8.9
    ]
  }
}
```

17. Select the target as SNS topic and choose the "guardduty-findings" topic.
18. Click Next, then Create rule.

You will now receive email notifications for GuardDuty findings with CRITICAL severity.
