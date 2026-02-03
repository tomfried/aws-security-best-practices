# Domain Setup for EC2 using self-managed everything
Note - is Cheaper but a bit more work

## 1. Make Site use Elastic IP
***Purpose - this sets your EC2 instance to a consistent public IP address so it won't change when it gets stopped and restarted.***
1. On AWS, under **"Network and Security"** section, select **"Elastic IPs"**.
2. Create one then edit it and associate it with your EC2 instance.
3. Visit your NEW elastic Ip Address at port 8000, ex: `http://44.123.45.67:8000/`

## 2a. Make app work at ex: `http://company.com:8000`
Setup Route53 hosted zone for domain to enable using your domain on EC2.
1. Create Hosted Zone. Make it **public** and set domain to base domain name ex: `company.com`.
2. Select Hosted Zone and verify you see a NS and SOA record type already.
3. Create new Record.
    - **Record type** A
    - **Value** YOUR_ELASTIC_UP_ADDRESS (ex. "44.123.45.67")
    - **TTL** "300" is fine
    - **Routing Policy:** Simple Routing
4. Save and after waiting a minute or two, confirm ex: `http://company.com:8000` works.

## 2b. Make app work also at ex: `http://www.company.com:8000`
Add Route53 record to same hosted zone for your domain to enable redirecting `www` to your domain.
1. Go to Route53, open up existing Hosted Zone: `company.com`
2. Create new Record.
    - **subdomain:** www
    - **Record type** A
    - **Value** YOUR_ELASTIC_UP_ADDRESS (ex. "44.123.45.67")
    - **TTL** "300" is fine
    - **Routing Policy:** Simple Routing
3. Save and after waiting a minute or two, confirm ex: `http://www.company.com:8000` automatically redirects people to `http://company.com:8000` and that still works.

## 3. Make app work at ex: `http://company.com`
In order to be able to run the project on a domain (ex. `http://company.com:8000`) but NOT at a port (`:8000`), we need to set up NGinx and generate some basic certs.
1. Setup NGINX
```shell
sudo yum install nginx -y
sudo mkdir /etc/pki/nginx
sudo chmod 755 /etc/pki/nginx
sudo cp ~/git-repos/hoa-app/app/nginx-dev.conf /etc/nginx/nginx.conf
sudo systemctl start nginx
sudo systemctl enable nginx
```
<details>
  <summary><b>Additional Resources</b> <i>(click to expand)</i></summary>

  * https://docs.aws.amazon.com/cloudhsm/latest/userguide/ssl-offload-configure-web-server.html

</details>

2. Add second Inbounding rule on AWS for port 80 to enable NGINX.
    - **Type:** HTTP
    - **Port:** 80
    - **CIDR Block:** 0.0.0.0/0
> [!NOTE]
> There should be 2 HTTP rules with a port of 80.

3. Generate a KEY and CSR for HTTPS.
```shell
sudo openssl req -newkey rsa:2048 -nodes -keyout /etc/pki/tls/private/site.key -out /etc/pki/tls/certs/site.csr
```
Enter the following (example):
```text
Country Name (2 letter code) [XX]:US
State or Province Name (full name) []:Kentucky
Locality Name (eg, city) [Default City]:PotatoLand
Organization Name (eg, company) [Default Company Ltd]:Company, Inc.
Organizational Unit Name (eg, section) []:.
Common Name (eg, your name or your server's hostname) []:company.com
Email Address []:company@gmail.com

Enter Challenge Password (20 characters max): ************
Enter Company Name again: Company, Inc.
```

4. Generated a temporary signed CERT from that KEY AND CSR (Needed for CA to eventually validate)
```shell
sudo openssl x509 -signkey /etc/pki/tls/private/company.key -in /etc/pki/tls/certs/company.csr -req -days 365 -out /etc/pki/tls/certs/company.crt
```

5. Generate CA key in order to generate CA public key.
```shell
sudo openssl genrsa -out /etc/pki/nginx/ca.key 2048
```

6. Generate CA public key file (`.pem`) - just to make Nginx happy.
```shell
sudo openssl req -x509 -new -nodes -key /etc/pki/nginx/ca.key -sha256 -days 1024 -out /etc/pki/nginx/ca.pem
```
Enter the following (example):
```text
Country Name (2 letter code) [XX]:US
State or Province Name (full name) []:Kentucky
Locality Name (eg, city) [Default City]:PotatoLand
Organization Name (eg, company) [Default Company Ltd]:Company, Inc.
Organizational Unit Name (eg, section) []:.
Common Name (eg, your name or your server's hostname) []:company.com
Email Address []:company@gmail.com

Enter Challenge Password (20 characters max): ************
Enter Company Name again: Company, Inc.
```

7. Copy certs to Nginx file
```shell
sudo cp /etc/pki/tls/certs/company.crt /etc/pki/nginx/server.crt
sudo cp /etc/pki/tls/private/company.key /etc/pki/nginx/server.key
# Do nothing for the ca.pem because it's already named and in correct location
```

8. Run gunicorn instead of python3 runserver to launch the app. To do this. refer to [Start Server]((#start-server) then verify system launches at that example, http://company.com

## 4. Make app work at ex: `https://company.com` (Enable HTTPS)
1. Replace self-signed cert with an officially signed cert.
```shell
sudo yum install certbot python3-certbot-nginx -y
sudo cp ~/git-repos/hoa-app/app/nginx-prod.conf /etc/nginx/nginx.conf
sudo cp ~/git-repos/hoa-app/app/company.com.conf /etc/nginx/conf.d
sudo certbot --nginx -d company.com
sudo certbot install --cert-name company.com
```
This will create 4 certs under `/etc/letsencrypt/live/company.com`
```
`[cert name]/privkey.pem`  : the private key for your certificate.
`[cert name]/fullchain.pem`: the certificate file used in most server software.
`[cert name]/chain.pem`    : used for OCSP stapling in Nginx >=1.3.7.
`[cert name]/cert.pem`     : will break many server configurations, and should not be used
                 without reading further documentation (see link below).
```
2. Do a `sudo certbot certificates` to confirm your certicate is showing up.

3. Lastly, [Restart Server](#start-server). Verify site works at `https://company.com`.

<details>
  <summary><b>Additional Resources</b> <i>(click to expand)</i></summary>

  * https://letsencrypt.org/how-it-works/
  * Ultimate Cert Guide: [Digital Ocean - OpenSSL Essentials](https://www.digitalocean.com/community/tutorials/openssl-essentials-working-with-ssl-certificates-private-keys-and-csrs).
  * Other HTTPS production guide: [Real Python  - Django Nginx Gunicorn](https://realpython.com/django-nginx-gunicorn/).
  * https://www.nginx.com/blog/using-free-ssltls-certificates-from-lets-encrypt-with-nginx/

</details>

## 5. Setup Certs to Automatically Renew
Because certbot certs only last 90 days, you will want to setup a cron to extend this.
1. Open up crontab files
```shell
crontab -e
```
2. Add the
```
0 12 * * * /usr/bin/certbot renew --quiet
```
Alternatively if you're a procrastinator:

<details>
  <summary><b>Manual Way</b> <i>(click to expand)</i></summary>

  ```
  sudo certbot certificates
  sudo certbot renew
  sudo certbot certificates
  ```

</details>
