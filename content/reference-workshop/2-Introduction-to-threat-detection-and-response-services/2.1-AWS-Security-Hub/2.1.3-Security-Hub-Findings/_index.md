---
title: "Security Hub - Findings"
date: "`r Sys.Date()`"
weight: 3
chapter: false
pre: " <b> 2.1.3 </b> "
---

## Security Hub - Findings

#### Findings in Security Hub

1. Click Findings in the left navigation to view all findings in Security Hub.
2. Use filters to narrow down the list. Click Add filter to get started.
3. Select Product name and type "GuardDuty" (case-sensitive), then click Apply.
4. Now you’ll see all findings from GuardDuty. There are many! Try adding another filter for high severity findings: select Severity label, choose is, and type HIGH (case-sensitive), then Apply.
5. Pick a finding and click its title to open the details pane. Expand all sections and take a few minutes to review the information—description, remediation, resource info, and more.
6. At the top of the details pane, click the finding ID link to view the full JSON for the finding. You can download this if you need to investigate further.
7. Close the JSON pop-up by clicking the X in the top right.
8. In the details pane, open the History tab to see a timeline of changes to the finding. This helps you track security risks and respond quickly.
9. Close the details pane by clicking the X, but stay on the Findings page.
10. Try to figure out which resources are generating the most findings. Remove all filters, then add a new filter: Resource type, choose is not, and type AwsAccount. Click Apply.
11. Add another filter: Record state, choose is, and type ACTIVE. Click Apply.
12. Click Add filters again, select Group by, and choose ResourceId. Click Apply. Now you’ll see findings grouped by resource.
