---
title: "Security Hub - Notifications"
date: "`r Sys.Date()`"
weight: 5
chapter: false
pre: " <b> 2.1.5 </b> "
---

## Security Hub - Notifications

#### Configure SNS topic

1. Go to Amazon SNS in the AWS console.
2. Click Topics in the left navigation.
3. Click Create topic.
4. Choose Standard type.
5. Name your topic "security-hub-findings".
6. Leave other settings as default and click Create topic at the bottom.

#### Subscribe to the topic

7. On the topic page, click Create subscription.
8. For Protocol, select email.
9. For Endpoint, enter your email address to receive notifications during the workshop. You can unsubscribe later.
10. Click Create subscription.
11. Check your email for a confirmation message and click the "Confirm subscription" link.

#### Create an EventBridge Rule to send findings to the topic

12. Navigate to Amazon EventBridge in the AWS console.
13. Click the Create rule button.
14. Name your rule "security-hub-findings" and click Next.
15. In the Event pattern section, click Edit pattern.
16. Add the following event pattern (JSON) to filter for CRITICAL severity findings:

```js
{
  "source": [
    "aws.securityhub"
  ],
  "detail-type": [
    "Security Hub Findings - Imported"
  ],
  "detail": {
    "findings": {
      "ProductName": [
        "Security Hub"
      ],
      "Severity": {
        "Label": [
          "CRITICAL"
        ]
      }
    }
  }
}
```

17. Click Next.
18. For Select a target, choose SNS topic.
19. For Topic, select security-hub-findings.
20. Click Next.
21. Click Next on the Configure tags - optional page.
22. Click Create rule on the Review and create page.

Keep an eye on your email for notifications from Security Hub findings.
