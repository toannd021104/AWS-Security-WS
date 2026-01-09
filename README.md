# AWS Workshop Hugo Site

This repository contains a Hugo-based static website for AWS security workshops and labs.

## Features
- All lab instructions rewritten in friendly, detailed English
- Security Hub, GuardDuty, Inspector, Detective, Partner/Aggregation labs
- Hugo theme and custom layouts
- Web counter and author info in footer

## How to build locally
```bash
hugo server -D
```
Then visit http://localhost:1313

## How to deploy with GitHub Actions
- Every push to the repository will trigger GitHub Actions to build and deploy the site to GitHub Pages automatically.
- No manual deployment needed.

## Custom domain
You can set a custom domain in GitHub Pages settings and point your DNS to GitHub.

## Author
Ngueyn Duc Toan

## Last updated
22/12/2025

---
For more details, see the workshop content in the `content/` folder.
