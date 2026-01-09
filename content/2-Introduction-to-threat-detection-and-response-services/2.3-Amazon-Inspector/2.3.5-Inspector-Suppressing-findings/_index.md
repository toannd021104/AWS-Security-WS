---
title: "Inspector - Suppressing findings"
date: "`r Sys.Date()`"
weight: 5
chapter: false
pre: " <b> 2.3.5 </b> "
---

## Inspector - Suppressing findings

Suppression rules help you focus on critical findings by hiding less important ones. Suppressed findings are still stored and published to Security Hub and EventBridge, but don’t appear in your list by default.

#### Set up a rule to suppress low severity findings in dev

1. Click Suppression rules in the left navigation.
2. Click Create Rule.
3. Name your rule "Low Severity - Dev Environment".
4. Add a description: "Suppress all findings with a low severity score for resources tagged with Development".
5. Under filters, select Severity and check Low. Click Apply.
6. Then click to add another filter and select Resource tag under EC2. Input "Environment" for the key and "Development" for the value. Click Apply.
7. Click Save.
8. You can view suppressed findings in the [**All Findings** page](https://us-east-1.console.aws.amazon.com/inspector/v2/home?region=us-east-1#/findings/all) by switching from Active to Suppressed in the dropdown.
