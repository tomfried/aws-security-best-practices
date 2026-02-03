# Production Hosting (ex. Django Python App)
1. Create EC2 and create new key pair.
2. Save **.pem RSA key** to _~/Documents/certificates_ folder.
3. Set permissions of the cert to chmod 600 for security.
  ```shell
  chmod 600 ~/Documents/certificates/company.pem
  ```
4. SSH to the new EC2 in AWS:
  ```shell
  # Example:
  ssh -i ~/Documents/certificates/company.pem ec2-user@ec2-3-21-456-789.us-east-2.compute.amazonaws.com
  ```
5. Install things:
  ```shell
  sudo dnf install git python3.12 -y
  python3.12 --version
  ```
6. Setup Cert for authentication with Github and download project.
  <details>
    <summary><b>Directions</b><i> [click to expand]</i></summary>
    ***Purpose, be able to pull in project from Github (If Github project is Private).***
    1. Find your "noreply" [email on Github](https://github.com/settings/emails). It should look like: `7778888+YOUR_GITHUB@users.noreply.github.com`
    2. On your terminal/PuTTy window on your EC2 instance, create new public/private key pair USING your noreply like so:
    ```shell
    cd ~/.ssh
    ssh-keygen -t rsa -b 4096 -C "7778888+YOUR_GITHUB@users.noreply.github.com"
    ```
    3. For a name enter ex: `git_rsa` then enter password when it prompts you.
    4. Once command finishes, open up the public (`.pub`) file by doing the following:
    ```
    less git_rsa.pub
    ```
    5. Copy the text from that file and paste it in a new [Github SSH key](https://github.com/settings/keys). Git clone should now work because your box's private key matches the public on Github.
    6. Start SSH agent and add your cert to confirm it works by doing the following:
    ```shell
    eval `ssh-agent -s`
    ssh-add ~/.ssh/git_rsa
    # Enter password
    ```
    7. Clone the project. Ex:
    ```shell
    cd ~
    git clone git@github.com:company/company-app.git
    ```
    8. If that worked, do the following to not need to do **step 6** ever again.
    ```shell
    touch ~/.ssh/config
    # ADD THE TEXT:    IdentityFile ~/.ssh/git_rsa
    echo "IdentityFile ~/.ssh/git_rsa" >> ~/.ssh/config
    # Save File
    chmod 600 ~/.ssh/config
    ```
  </details>

7. Setup Database.
  <details>
    <summary><b>7a. within EC2</b><i> [click to expand]</i></summary>
    1. Install mariadb, start and the enable the service on startup, then open mysql
      ```shell
      sudo dnf install mariadb105 mariadb105-server -y
      sudo systemctl start mariadb
      sudo systemctl enable mariadb
      sudo mysql_secure_installation
      # Enter "Y" to everything but changing root password and unix authentication and set really good password like "dyPbvpNSfMlZPYyRQXlp" YOU CAN REMEMBER.
      ```
    2. Add Environment Variables that match the RDS values.
      ```shell
      sudo vi /etc/environment
      #Copy and Paste following into file, ex:
      CUSTOM_DOMAIN=company.com
      SECRET_KEY=abcd7836370&$$@^(*dhpoooooqnnv)
      DB_USER=root
      DB_PASSWORD=dyPbvpNSfMlZPYyRQXlp
      # But include the password you just set.
      ```
    3. Do a reboot so the changes can take affect.
      ```shell
      sudo reboot
      ```
    4. After a few minutes, SSH back into the project and check to see if the environment variables exist. Ex:
      ```shell
      env
      echo $DB_USER
      echo $DB_PASSWORD
      ```
    5. Setup database for the project:
      ```shell
      mysql -P 3306 -u $DB_USER --password=$DB_PASSWORD
      ```
      ```mysql
      CREATE DATABASE djangodb CHARACTER SET UTF8;
      use djangodb;
      exit
      ```
  </details>
  <details>
    <summary><b>7b. in RDS (extra $16/month but more scalable)</b><i> [click to expand]</i></summary>
    1. Setup RDS. [Example Guide](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/CHAP_GettingStarted.CreatingConnecting.MariaDB.html)
      - type = ex. MariaDB
      - name = "djangodb"
      - master_username = "admin"
      - password = add a crazy password ex. 18-32 characters and write it on a post-it note.
      - Connect it to your EC2.
    2. Add Environment Variables that match the RDS values.
      ```shell
      sudo vi /etc/environment
      #Copy and Paste following into file, ex:
      CUSTOM_DOMAIN=company.com
      SECRET_KEY=abcd7836370&$$@^(*dhpoooooqnnv)
      DB_HOSTNAME=djangodb.abcd1234.us-east-1.rds.amazonaws.com
      DB_USER=admin
      DB_PASSWORD=dyPbvpNSfMlZPYyRQXlp
      ```
    3. Do a reboot so the changes can take affect.
      ```shell
      sudo reboot
      ```
    4. After a few minutes, SSH back into the project and check to see if the environment variables exist. Ex:
      ```shell
      env
      echo $DB_HOSTNAME
      echo $DB_USER
      echo $DB_PASSWORD
      ```
    5. Install MaraiDB client to see what is in AWS RDS and make the database to begin the project:
      ```shell
      # Install the latest available MariaDB client from Amazon Linux repos
      sudo dnf install -y mariadb105
      mysql -h $DB_HOSTNAME -P 3306 -u $DB_USER --password=$DB_PASSWORD
      CREATE DATABASE djangodb CHARACTER SET UTF8;
      ```
  </details>
8. Create virtual environment (for now)
  ```shell
  cd ~/company-app
  python3.12 -m venv eenv
  source eenv/bin/activate
  python3.12 -m pip install -r requirements.txt
  ```
9. Create Database.
  ```shell
  cd ~/company-app/app
  python3.12 manage.py migrate
  python3.12 manage.py createsuperuser
  # ex. username = admin
  # ex. email = {YOURS}
  # ex. password = idk
  ```
10. Install and setup REDIS.
  ```shell
  sudo dnf install redis6 -y
  sudo systemctl start redis6
  sudo systemctl enable redis6
  ```
11. Start app.
  ```shell
    python3.12 manage.py runserver 0.0.0.0:8000
    # But use the following for PROD:
    # nohup gunicorn --bind 0.0.0.0:8000 config.wsgi:application &
  ```
12. Add Inbounding rule on AWS for port 8000 to enable HTTP. ([pictures explaining how to do it](https://stackoverflow.com/questions/34577076/access-django-app-on-aws-ec2-host))
      - **Type:** Custom TCP
      - **Port:** 8000
      - **CIDR Block:** 0.0.0.0/0
13. Visit your public IP Address at port 8000, ex: `http://12.345.67.89:8000/`
