---
title: "Security Hub - Overview"
date: "`r Sys.Date()`"
weight: 1
chapter: false
pre: " <b> 2.1.1 </b> "
---

## Security Hub - Overview

#### Security Standards

1. Start by heading to the AWS Security Hub summary page. This is your main dashboard for security insights.
2. On the Summary page, you'll easily spot which standards are active in your account. You’ll also see how many controls have passed or failed for each standard, and a handy percentage showing your overall compliance status.
3. To explore further, click on Security standards in the left navigation menu.
4. Good news: the standards were already enabled for you using CloudFormation when the workshop was set up!
5. Let’s take a closer look at a specific standard. Click View Results for NIST Special Publication 800-53 Revision 5 to see more details.

#### Controls

6. At the top, you can switch between tabs to view only "Passed", "Failed", or "Disabled" controls. You can also use the search bar to filter controls by ID. Try searching for ID = EC2.13.
7. Click on the control titled "Security groups should not allow ingress from 0.0.0.0/0 to port 22." This will show you all resources checked for this control and their current status—some may have FAILED, others PASSED.
8. For every check, AWS Security Hub provides helpful remediation instructions. Click the Remediation instructions link at the top to open guidance in a new tab.

#### Investigating a misconfigured resource

9. Return to the Security Hub control you were looking at. For one of the **FAILED** resources open **Config rule** in the **Investigate** column. This will display links that will take you to AWS Config to view the configuration timeline for this resource or the overall config rule that performed the evaluation on this resource.
   ![VPC](/images/2/2.1-AWS-Security-Hub/2.1.1-security-hub-overview/s9.png)

10. Click **Configuration timeline**. This opens AWS Config, and shows the resource configuration timeline for this security group. From here, you can investigate the point in time when the resource became non-compliant by reviewing the timestamped events. If you want, take a minute to expand a couple of the events.
    ![VPC](/images/2/2.1-AWS-Security-Hub/2.1.1-security-hub-overview/s10.png)
11. Expand a **CloudTrail Event** listed.
    ![VPC](/images/2/2.1-AWS-Security-Hub/2.1.1-security-hub-overview/s11.png)
12. Then click **CloudTrail** under View event. This opens up that event, logged in CloudTrail. AWS CloudTrail monitors and records account activity across your AWS infrastructure, giving you control over storage, analysis, and remediation actions.
    ![VPC](/images/2/2.1-AWS-Security-Hub/2.1.1-security-hub-overview/s12.png)

#### Consolidated controls

13. Return to Security Hub and open General settings page from the navigation panel. https://console.aws.amazon.com/securityhub/home?#/settings/general
    ![VPC](/images/2/2.1-AWS-Security-Hub/2.1.1-security-hub-overview/s13.png)
14. At the top of the page, you should see that both "Auto-enable new controls in enabled standards" and "Consolidated control findings" is enabled.

15. Click Controls from the navigation panel. This page shows a similar view to what you reviewed at the standard level, but here you see the full list of consolidated controls. This makes it easier to see how many individual controls in your account are passing, failing, or in some other state. Additionally, with consolidated findings enabled, controls included in multiple enabled standards no longer generate a finding per standard (duplicates). This helps manage the number of findings in your environment and removes potential confusion around duplicate findings.

![VPC](/images/2/2.1-AWS-Security-Hub/2.1.1-security-hub-overview/s15.png)

#### Insights

16. In the navigation on the left, click **Insights**.
    ![VPC](/images/2/2.1-AWS-Security-Hub/2.1.1-security-hub-overview/s16.png)
17. In the search bar, input **"severity"** and select the insight named **24. Severity by counts of findings**.
    ![VPC](/images/2/2.1-AWS-Security-Hub/s17.png)
18. Security Hub offers several built-in managed insights. You cannot modify or delete managed insights. **24. Severity by counts of findings** is one of the managed insights. Take a minute to review the number of findings in the environment. Select a **Severity Label** (e.g. Critical) to see the underlying finding(s).
    ![VPC](/images/2/2.1-AWS-Security-Hub/s18.png)
