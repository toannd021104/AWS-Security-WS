---
title: "Cross-region finding aggregation"
date: "`r Sys.Date()`"
weight: 4
chapter: false
pre: " <b> 3.4. </b> "
---

## Cross-region finding aggregation

With cross-Region aggregation, you can collect findings, updates, insights, and scores from multiple regions into one aggregation region. Manage all your data from this central location.

#### Set up cross-region finding aggregation

1. If prompted, click Configure finding aggregation. Or open the Regions page under Settings.
2. Click Configure finding aggregation.
3. Select US East (N. Virginia) as your Aggregation Region.
4. Select all available regions.
5. Click Save to enable cross-region aggregation. Note: This erases aggregated scores and control data, but findings remain. Scores will reappear after new calculations.

#### Turn on AWS security services in linked regions

Enable all needed AWS security services in your linked regions using CloudFormation. Once enabled, findings from those regions will be aggregated in your central region.
