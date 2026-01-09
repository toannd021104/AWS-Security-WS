---
title: "Building your own Security Hub integration"
date: "`r Sys.Date()`"
weight: 5
chapter: false
pre: " <b> 3.5. </b> "
---

## Building your own Security Hub integration

You can build custom solutions to send Security Hub findings to places like Slack. For example, set up a custom action to send finding details to a Lambda function (via EventBridge), which formats the message and posts it to a Slack channel.

#### Setup Slack workspace

1. Create a new Slack workspace using the provided link.
2. Make a channel called "security-hub-alerts" for notifications. Set it to public.

#### Setup Slack application for integration

3. Go to the Slack API site and click Your apps, then Create an App.
4. Choose From scratch, name your app "security-hub-to-slack", and select your workspace.
5. Click Create App.
6. Open Incoming Webhooks and turn them ON.
7. Click Add New Webhook to Workspace, select **#security-hub-alerts**, and click Allow.
8. Copy the Webhook URL for later use.

#### Create Custom Action in Security Hub

Slack is setup. Now you need to configure the Security Hub custom action, EventBridge rule, and Lambda function to push findings to your Slack app (via the webhook).

9. Return to Security Hub and open the Custom actions page.
10. Click Create custom action.
11. Name it "Send to Slack", describe it, and set the Custom action ID to "SendToSlack".
12. Copy the Custom action ARN for later use.

#### Create the EventBridge Rule

13. Navigate to Amazon EventBridge and click Create rule.
14. Name and describe your rule, then click Next.
15. Leave Event source as AWS events or EventBridge partner events.
16. Set AWS Service to Security Hub.
17. Choose Security Hub Findings – Custom Action for Event Type.
18. Select the Specific custom action ARN(s) radio button and enter your custom action ARN.
19. Click Next.
20. Choose Lambda function as the target and select security-hub-to-slack.
21. Click Next, then Next again, and finally Create rule.

#### Customize the Lambda function to send messages to your Slack channel

22. Go to the Lambda console, open the security-hub-to-slack function, and navigate to Configuration > Environment variables.
23. Edit the variables: set slackChannel to "security-hub-alerts" and webHookUrl to your copied webhook.
24. Click Save.

#### Test the Integration

25. In Security Hub, go to the Findings page.
26. Select a finding and choose Send to Slack from the Actions dropdown.
27. Check the #security-hub-alerts channel in Slack for the new message.

#### Automate the slack messages

**Problem Statement**:
Your manager is concerned about the scalability of the appraoch you implemented because it requires you to manually select the findings to be sent to Slack. You have been instructed to automate this so findings that have MEDIUM or HIGH severity labels are automatically sent to Slack via the integration.
Can you figure out what changes need to be made in order to accomplish this objective?

**Hint**: \
6. Select **Specific Severity label(s)** and in the dropdown, pick **MEDIUM** and **HIGH**. That's it. Continue through the same steps as before
![VPC](/images/3/3.5/h6.png)

7. Click **Next**.

8. On the Select target(s) step, select Lambda function from the Select a target dropdown. Then select security-hub-to-slack from the Function dropdown. Click Next.

![VPC](/images/3/3.5/h6-s8.png)

9. On the Configure tags step, click Next.

10. On the Review and create step, click Create rule.
    ![VPC](/images/3/3.5/h6-s10a.png)
    ![VPC](/images/3/3.5/h6-s10b.png)
