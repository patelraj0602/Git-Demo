# Hosting Website Using VM

## 1) Basic Setup
    - You can check the status of your firewall using : sudo ufw status
    - Configuring UFW to allow incoming SSH connections ; To allow pubkey based authentication use : sudo ufw allow ssh
    - Finally to enable ufw use : sudo ufw enable
&nbsp;
## 2) User Management
### The following are some of the different types of user account which you can create
#### - Student Account 
    - To make the student user without sudo permissions : sudo adduser student
    - Cmd will prompt you to add password; Type the password for your user account. This account which got created does not have sudo permissions.
#### - Administrator Account
    - To make the student user without sudo permissions : sudo adduser adminstrator
    - Type in password for your admin account; Now to config you admin account so that it does not ask for password for using sudo : sudo usermod -aG sudo administrator
#### - Audit Account
    - Audit accounts are the accounts which does not ask passwords for using sudo
    - This type of account is created by default when you create your VM
    
#### - Application Account
    - These type of account does not have home directory.
    - To create this type of account use : sudo useradd -M application
&nbsp;
## 3) Website Management
### Step1 : Install nginx
    Reference used to do this step : https://flaviocopes.com/nginx-https-certbot/
    - Install Cerbot and Certbot Nginx package : sudo apt-get install certbot python3-certbot-nginx
    - Edit server_name field in default file(/etc/nginx/sites-available/default) to your domain name 
        server_name raj.ssl.airno.de
    - Reload nginx using : sudo systemctl reload nginx
    Note : Make sure to give permissions to nginx, nginx full, nginx http, nginx https, openssh using ufw which we discussed earlier.
### Step2 : Setup https certs for domain
    Generate the https certs using certbot using : sudo certbot --nginx -d raj.ssl.airno.de
### Step3 : Hosting a static html page on your domain name
    There are couple of ways to do this task:
    Method 1 :
    - Dump your index.html file in your vm using scp command
    - And then change the path to this index.html in your default file(/etc/nginx/sites-available/default) to render this html file by default when someone visits your website.
    Method 2(Which I used) : 
    - The html file which gets rendered by default is present in (/var/www/html)
    - Change the html present in it to display your name.
### Step4 : Reverse proxy an app
    1) Verify the sha256 signature using : shasum -a 256 '<path to app>'
    2) To setup an nginx reverse proxy we need add some code to our default file(/etc/nginx/sites-available/default) to forward the requests from the client to our backend. 
    This is my default file setting
            location /app1/{
                proxy_pass http://localhost:8008/app1/;
                proxy_http_version 1.1;
                proxy_cache_bypass $http_upgrade;
                proxy_set_header Upgrade $http_upgrade;
                proxy_set_header Connection        "upgrade";
                proxy_set_header Host              $host;
                proxy_set_header X-Real-IP         $remote_addr;
                proxy_set_header X-Forwarded-For   $proxy_add_x_forwarded_for;
                proxy_set_header X-Forwarded-Proto $scheme;
                proxy_set_header X-Forwarded-Host  $host;
                proxy_set_header X-Forwarded-Port  $server_port;
            }
            
            location /app2/{
                proxy_pass http://localhost:8008/;
                proxy_http_version 1.1;
                proxy_cache_bypass $http_upgrade;
                proxy_set_header Upgrade $http_upgrade;
                proxy_set_header Connection        "upgrade";
                proxy_set_header Host              $host;
                proxy_set_header X-Real-IP         $remote_addr;
                proxy_set_header X-Forwarded-For   $proxy_add_x_forwarded_for;
                proxy_set_header X-Forwarded-Proto $scheme;
                proxy_set_header X-Forwarded-Host  $host;
                proxy_set_header X-Forwarded-Port  $server_port;
            }
    3) Add port 8008 using ufw command which I discussed above; Make sure to close port 8008 on my VM under networking section because the client should not be able to access the application directly by typing https://raj.ssl.airno.de:8008
    
    Note : Don't forget to run your application(app) before using the url(https://raj.ssl.airno.de/app1/)
    To run the application on your vm use the following command:
        1. chmod a+x ./app
        2. chmod +x app
        3. ./app&
&nbsp;
## 4) Setting Up MySql
### - Step1 : Install MySql server
    To install mysql-server simple type this command : sudo apt-get install mysql-server
### - Step2 : Creating Database And User using MySql
    Note : Before executing the following commands make sure that u are inside mysql
    - Creating a database called onboarding : CREATE DATABASE onboarding;
    - Creating a user inside mysql : CREATE USER 'Raj Patel'@'raj.ssl.airno.de';
    - Granting all the privileges to "Raj Patel" on "onboarding" : GRANT ALL PRIVILEGES ON onboarding.* TO 'Raj Patel'@'raj.ssl.airno.de' IDENTIFIED BY 'password';
    We are good to go now this mysql will not be accessible from outside the vm because we made a database on our vm using mysql-server.
    
        
        
        
