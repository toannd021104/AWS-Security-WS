---
title: "Centralizing findings from AWS security services"
date: "`r Sys.Date()`"
weight: 1
chapter: false
pre: " <b> 3.1. </b> "
---

Security Hub automatically collects and consolidates findings from AWS security services enabled in your environment, such as intrusion detection findings from Amazon GuardDuty, vulnerability scans from Amazon Inspector, Amazon Simple Storage Service (Amazon S3) bucket policy findings from Amazon Macie, publicly accessible and cross-account resources from IAM Access Analyzer, and resources lacking WAF coverage from AWS Firewall Manager.

This aggregation provides a view of security and compliance across your AWS environment. AWS Security Hub aggregates security findings from dozens of AWS security services and APN products in a single place and format, the AWS Security Finding Format (ASFF). Security Hub doesn't retroactively detect and consolidate security findings that were generated before you enabled Security Hub. All findings are stored in Security Hub for 90 days after last update date.

The Integrations page in the AWS Management Console provides access to all of the available AWS and third-party product integrations. The AWS Security Hub API also provides operations to allow you to manage integrations. For AWS services, Security Hub automatically enables the integration, and you can optionally disable each integration.

## Centralizing findings from AWS security services

Security Hub brings together findings from all your AWS security services—like GuardDuty, Inspector, Macie, Access Analyzer, and Firewall Manager—so you get a complete view of your security and compliance. Findings are stored for 90 days and shown in a unified format.

#### Centralizing findings from AWS security services

1. Go to Security Hub and open the Summary page for an overview of your account’s security status.
2. Review the insights to find common threat types. These are aggregated from services like GuardDuty.
3. Open the Integrations page to see all available integrations. Many, like GuardDuty and Inspector, are enabled automatically, but you need to turn on each service to generate findings.
4. You can enable or disable integrations as needed. Try disabling the Firewall Manager integration, but click Cancel to keep it active.
5. To see findings from a specific service, click See findings for that integration. This adds a filter so you only see those findings.
6. Adjust filters to change the list of findings you’re viewing.

#### Searching and reviewing security findings

7. To see all findings, remove filters and Group By options at the top of the page.
8. Try adding a filter for high severity findings from GuardDuty.
9. From the dropdown, select Severity label and choose is and then input HIGH. This is case-sensitive. Click Apply.
   ![VPC](/images/3/3.1/s10a.png)
   Result:
   ![VPC](/images/3/3.1/s10b.png)
   Choose one finding to see more information
   ![VPC](/images/3/3.1/s10c.png)
10. Click Add filters in the search bar again. This time, select Product name and choose is and then input GuardDuty. Click Apply.

11. Pick one of the findings and click on the title. This opens the finding details pane. Expand all the sections and take a few minutes to review the information here. You can see the description, a link to remediation instructions, information about the resource, and more.

12. In the finding details pane click the finding ID link at the top of the pane to display the complete JSON for the finding. Security Hub processes findings using a standard findings format called the AWS Security Finding Format (ASFF), which eliminates the need for time-consuming data conversion efforts. The finding JSON can be downloaded to a file if ever needed for further investigation.
    ![VPC](/images/3/3.1/s13.png)

13. Close out of the JSON pop-up by clicking the X in the top right.

14. At the top of the finding details, open the History tab to view a chronological list of all changes that have been made to the finding. The transparency of finding history helps you identify potential security risks more quickly and take proactive steps to mitigate them.

15. Close the finding details pane by clicking the X in the top right, but stay on the findings page.

16. Let's try to understand what resources in our environment are generating the most findings. Remove all the filters. Then add a new filter, select Resource type and choose is not and then input AwsAccount. Click Apply.
    ![VPC](/images/3/3.1/s17a.png)
    Result:
    ![VPC](/images/3/3.1/s17b.png)
17. Add another filter. Select Record state, choose is, and then input ACTIVE. Click Apply.
    ![VPC](/images/3/3.1/s18.png)
18. Click Add filters in the search bar again. This time, select Group by and choose ResourceId. Click Apply. This list shows you the number of security findings per resource. This can help you prioritize work by the resources in your environment with the most security issues.
    ![VPC](/images/3/3.1/s19.png)
    Result:
    ![VPC](/images/3/3.1/s19b.png)

#### Generating insights from aggregated findings

19. If you find the list of number of security findings per resource beneficial, you can save it as a custom Insight for future reference. Click Create insight in the top right of the screen.

20. Name the insight "0. Resources by counts of findings". Then click Create insight.
    ![VPC](/images/3/3.1/s20.png)
21. In the navigation on the left, click Insights. A Security Hub Insight is a collection of related findings defined by an aggregation statement and optional filters. An insight identifies a security area that requires attention and intervention. Security Hub offers several managed (default) insights that you can't modify or delete. You can also create custom insights to track security issues unique to your AWS environment and usage.

22. The first Insight listed is the one you just created. Click on the title 0. Resources by counts of findings. Here you have the same view as before plus generated graphs. We'll dive deeper into Insights later.
    ![VPC](/images/3/3.1/s23.png)
23. Return to the Insights page using the navigation on the left.

24. Take a few minutes to review the other AWS-provided Insights. Filter for insight severity.

25. Click on 24. Severity by counts of findings.

26. Select a Severity Label (e.g. Critical) to see the underlying finding(s).

![VPC](/images/3/3.1/s26.png)
Result:
![VPC](/images/3/3.1/s26b.png)
