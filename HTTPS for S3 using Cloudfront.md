# HTTPS Domain Setup for S3 using Cloudfront

## Monthly Cost
* ~$0.00 for Cloudfront
* ~$0.04 in S3
* $0.50 for Route53
* ~$1.25 for domain (billed as $15/year for .com domain)
**TOTAL = <$2/month** (or $0.55/month with a once a year $15 charge)

## Setup
1. Go to **Route53** and register new domain (ex. `mydomain.com`). If you think you will keep it, recommend setting up **automatic renewal**.
2. Go to **S3** and make new bucket that perfectly matches that domain name: `mydomain.com`. Enable and configure it with the following:
  * TBD
  * TBD
3. On that `mydomain.com` S3 bucket, upload all of the contents of your website. Test it worked and can be accessed from the internet through the link provided on the **Properties** tab, ex: http://mydomain.com.s3-website-us-east-1.amazonaws.com. If it does not work, check the settings of the bucket.
  * TBD
  * TBD
4. *OPTIONALLY*, if you desire a **"www."** option as well that will redirect to your "mydomain.com" in **S3** create another called `www.mydomain.com` with the following settings:
5. If you wanted HTTP only you could just update Route53 records to point to the S3 buckets, but for HTTPS, next go to **Cloudfront**.
6. Create a new **Cloudfront Distribution** with the following settings:
  * Scroll to the very bottom and select **"Pay as you go"** NOT "Free". Can't explain why, it just poses weird boundaries for no reason; it's still free.
  * **Distribution Name:** "mydomain.com" *(must be identical to the domain)*
  * **Distribution Type:** "Single website or app"
  * **Domain:** Set and connect to your Route53 one of the same name.
  * **Origin:** S3 > Then select the `mydomain.com` S3 bucket
  * **Web Application Firewall (WAF):** `Disabled` - it costs ~$15/month when it's all said and done and has nothing to really protect for static websites so don't bother.
7. After 1-2 minutes check the URL provided (ex. `dl212ofhs68.cloudfront.net`) to see if it works. If it does, it means your Cloudfront distribution was setup correctly! If not, check what settings you provided.
8. Once you can confirm it does work, test accessing it at your actual domain in the browser: https://mydomain.com and also http://mydomain.com. If it does not work, manually create or select the button on your Cloudfront distribution to populate these records in Route53 automatically.

If at any point it still does not connect or you guys errors with the last step, ensure there is not already an Alias (A) record for `www.mydomain.com` and `mydomain.com`. The alias should point TO the cloudfront which directs to the S3 behind the scenes, do not alias directly to the S3.
