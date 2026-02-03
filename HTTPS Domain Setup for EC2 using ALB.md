# Domain Setup for EC2 Using ALB (Application Load Balancer)
## 1. Make Site use Elastic IP
***Purpose - this sets your EC2 instance to a consistent public IP address so it won't change when it gets stopped and restarted.***
1. On AWS, under **"Network and Security"** section, select **"Elastic IPs"**.
2. Create one then edit it and associate it with your EC2 instance.
3. Visit your NEW elastic Ip Address at port 8000, ex: `http://44.123.45.67:8000/`

## 3. Make app work at ex: `https://company.com`
In order to be able to run the project on a domain (ex. `http://company.com:8000`) but NOT at a port (ex. `:8000`), and all using AWS services, do the following:

1. Create an **ACM** for HTTPS (Amazon Certificate Manager). Cert will auto renew too.
2. Select to Create a record for this in Route53.
3. Create **"Target Group"** pointing to your EC2 and port it is running on. Set Health equal to "/" unless you have a ex: `/health` endpoint.
3. Create **Load Balancer** > of type "ALB" (Application Load Balancer)
4. Create 2 listeners:
  - set port to "HTTPS:443" and set to **Target Group** source (for app to show up at HTTPS)
  - set port to "HTTP:80" and set to "HTTPS:443" (for app to redirect HTTP to HTTPS)
5. Test DNS that is automatically generated.
6. Go to **Route 53** and set up an **A** record on that domain that points to the ALB Load Balancer for the root URL.
7. Test the site at https://company.com.
8. If it works, adjust the Security Groups of your EC2 and ALB to the following. If it didn't work, try opening all ports and all types of traffic on both, then correct to the following afterwards to tighten for security.
**ALB Security Group**
  - Inbound: HTTPS (443) -> all IPv4 Traffic
             HTTP (80) -> all IPv4 Traffic
  - Outbound: All

**EC2 Security Group**
  - Inbound: Custom TCP (8000) -> The ALB Security Group
             SSH (22) -> my IP Address
  - Outbound: All

9. OPTIONALLY, **support "www"** redirecting to that URL by going to **Route53**, creating identical **A** record as before in step 6 but with "www." as the subdomain.
