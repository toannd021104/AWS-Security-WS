---
title: "Inspector  - Findings"
date: "`r Sys.Date()`"
weight: 3
chapter: false
pre: " <b> 2.3.3 </b> "
---

## Inspector - Findings

#### Filter findings

1. Go to Findings: All Findings.
2. Use the Filter criteria input at the top to add filters. This helps you focus on findings that matter most. Try filtering by resource tag: select Resource tag, enter Key "Environment" and Value "Development", then Apply.
3. Remove the filter by clicking Clear filters when done.

#### Review a network reachability finding for EC2

4. Add a filter for Type: select Network Reachability and Apply.
5. Click a finding title to view its report. You’ll see Severity, open port range, account info, and network paths.

#### Reviewing a code vulnerability finding for Lambda

9. Remove all the filters by clicking **Clear filters**. Click the field, **Add filter**. Select **Type** and then select **Code Vulnerability**. Click **Apply**.

10. Click the title of finding, **CWE-117,93 - Log injection**, to view the finding report.

11. The finding states "User-provided inputs must be sanitized before they are logged. An attacker can use unsensitized input to break a log's integrity, forge log entries, or bypass log monitors." The details in the report include the information about the resource and suggested remediation. Read the suggested remediation. As a challenge, come back later if you have time and try to remediate the issue WITHOUT deleting the function.

12. Close out of the finding and remove all the filters by clicking **Clear filters** again.
