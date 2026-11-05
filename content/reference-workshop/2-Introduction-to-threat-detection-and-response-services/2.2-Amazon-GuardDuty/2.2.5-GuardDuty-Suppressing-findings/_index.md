---
title: "GuardDuty - Suppressing findings"
date: "`r Sys.Date()`"
weight: 5
chapter: false
pre: " <b> 2.2.5 </b> "
---

## GuardDuty - Suppressing findings

#### Set up a rule to suppress unwanted findings

1. In GuardDuty, open the Findings page.
2. Click Suppress Findings.
3. Add your first filter: select Finding Type and type "Recon:EC2/Portscan". Click Apply.
4. Check the preview to see which findings will be suppressed. To avoid missing important alerts, add a second filter: select Instance Tag Value and type "Scanner". Click Apply.
5. Now only findings from your vulnerability assessment tool will be suppressed.
6. Name your rule "VulnerabilityAssessmentTool".
7. Add a description: "Suppress findings from Vulnerability Assessment Tool."
