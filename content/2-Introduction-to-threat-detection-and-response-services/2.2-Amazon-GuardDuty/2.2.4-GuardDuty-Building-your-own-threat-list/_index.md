---
title: "GuardDuty - Building your own threat list"
date: "`r Sys.Date()`"
weight: 4
chapter: false
pre: " <b> 2.2.4 </b> "
---

## GuardDuty - Building your own threat list

#### Implementing a threat list with a test address

1. Go to Amazon S3 in the AWS console.
2. Find the bucket with "-threatlistbucket-" in its name and click it.
   ![VPC](/images/2/2.2-Amazon-GuardDuty/2.2.4-GuardDuty-Building-your-own-threat-list/s2.png)

Here is the bucket that the AWS Workshop account has setup
![VPC](/images/2/2.2-Amazon-GuardDuty/2.2.4-GuardDuty-Building-your-own-threat-list/s2b.png) 3. Upload your custom threat list file to this bucket.
![VPC](/images/2/2.2-Amazon-GuardDuty/2.2.4-GuardDuty-Building-your-own-threat-list/s3.png)

4. Copy the S3 URI of your uploaded file for later use. To find the URI, open the bucket, click to open the file, and copy the S3 URI.
   ![VPC](/images/2/2.2-Amazon-GuardDuty/2.2.4-GuardDuty-Building-your-own-threat-list/s4.png)

5. Now, add your threat list to GuardDuty: go back to GuardDuty and select Lists in the left navigation.

6. Click Add a threat IP list.
   ![VPC](/images/2/2.2-Amazon-GuardDuty/2.2.4-GuardDuty-Building-your-own-threat-list/s7.png)

7. Give your list a friendly name like "Test Threat List".

8. Under location type the S3 URI for the uploaded file that you recorded earlier.

9. Select **Plaintext** as the Format.
   ![VPC](/images/2/2.2-Amazon-GuardDuty/2.2.4-GuardDuty-Building-your-own-threat-list/s10.png)

10. Click **I Agree** and then click **Add list**

11. You will need to activate the list once it has been added to GuardDuty. Select the newly added list. Click **Actions**, and then select **Activate**.
    ![VPC](/images/2/2.2-Amazon-GuardDuty/2.2.4-GuardDuty-Building-your-own-threat-list/s12.png)

12. A green status banner will appear saying "Successfully updated the list...".

![VPC](/images/2/2.2-Amazon-GuardDuty/2.2.4-GuardDuty-Building-your-own-threat-list/s13.png)
