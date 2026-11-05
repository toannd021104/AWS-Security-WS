---
title: "GuardDuty - Findings"
date: "`r Sys.Date()`"
weight: 2
chapter: false
pre: " <b> 2.2.2 </b> "
---

## GuardDuty - Findings

A GuardDuty finding is like a security alert for your AWS environment. Whenever GuardDuty spots something odd or potentially risky, it creates a finding for you to review.

#### Review a GuardDuty finding

1. Go to the Findings section in Amazon GuardDuty by clicking Findings in the left navigation.
2. Pick any finding from the list by clicking its row. This opens a summary on the right side. Each finding has details based on its type.
3. Take a moment to review the details of the finding you selected.

#### Understanding GuardDuty finding severity

4. Check the Severity of your chosen finding to see how critical it is.
5. Want to see the raw data? Click the Finding ID at the top to view or download the JSON for the finding.
6. When you’re done, close the summary by clicking the X in the top right.

#### Searching and Filtering GuardDuty Findings

7. Click on the Search bar where it says **Add filter criteria**.

8. Type **Severity** in the search bar and click on **Severity**, which then opens to a sub-menu with Low, Medium, and High options.

9. Check the box for **High** and click **Apply**. This will then update the list of displayed findings accordingly.
   ![VPC](/images/2/2.2-Amazon-GuardDuty/2.2.2-GuardDuty-Findings/s9a.png)

![VPC](/images/2/2.2-Amazon-GuardDuty/2.2.2-GuardDuty-Findings/s9b.png)

#### Managing GuardDuty findings

10. With a finding selected, click the **Actions** dropdown (top right of the page). Click **"Archive"** to archive the finding.

![VPC](/images/2/2.2-Amazon-GuardDuty/2.2.2-GuardDuty-Findings/s10.png)

11. Archiving the finding will hide it from the list of Current findings. To view it, click the **"Current"** dropdown, and select **"Archived"** to see the finding you just archived.

![VPC](/images/2/2.2-Amazon-GuardDuty/2.2.2-GuardDuty-Findings/s11a.png)
Result:
![VPC](/images/2/2.2-Amazon-GuardDuty/2.2.2-GuardDuty-Findings/s11b.png)

12. To unarchive the finding, select it, and then click the **Actions** dropdown again (top right of the page). This time, click **"Unarchive"** to unarchive the finding.

![VPC](/images/2/2.2-Amazon-GuardDuty/2.2.2-GuardDuty-Findings/s2.png)
