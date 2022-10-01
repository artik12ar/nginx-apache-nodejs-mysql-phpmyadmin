## Introduction

Setting up nginx as reverse proxy that forward traffic to apache web server to serve PHP and nodejs application using different port. Self reference only easy for me to refer next time when needed.

## Requirements
- nginx
- php (Using php 7.4)
- apache
- nodejs simple application run on `port: 3000`
- PHP application run on `port :8080`
- Mysql
- phpmyadmin

## Installations

### Install Apache

Update local package index
```
sudo apt update
```

Install apache2 package
```
sudo apt install apache2
```

Change default port from `80` to `8080`
```
sudo vim /etc/apache2/ports.conf

# change Listen 80 into Listen 8080. Save and close vim editor
```

Restart apache2
```
sudo systemctl restart apache2
```

stop apache2
```
sudo systemctl stop apache2
```

Check status
```
sudo systemctl status apache2
```

Reload apache2
```
sudo systemctl reload apache2
```

Disable apache2 from startup boot
```
sudo systemctl disable apache2
```

Enable apache2 to start on startup boot
```
sudo systemctl enable apache2
```

### Install PHP (7.4)

```
sudo apt-get install software-properties-common
sudo add-apt-repository ppa:ondrej/php
sudo apt-get update
sudo apt-get install -y php7.4
```

Check version
```
php -v
```

Install common modules
```
sudo apt-get install php7.4-mysql php7.4-curl php7.4-zip php7.4-json php7.4-cgi php7.4-xsl
```

Change common config
```
sudo vim /etc/php/7.4/apache2/php.ini

## Find this setting and change to
file_uploads = On
allow_url_fopen = On
memory_limit = 256M
upload_max_filesize = 100M
max_execution_time = 360
```

### Install Nginx

Update local package index
```
sudo apt update
```

Install nginx package
```
sudo apt install nginx
```

Restart nginx
```
sudo systemctl restart nginx
```

Stop nginx
```
sudo systemctl stop nginx
```

Check status
```
sudo systemctl status nginx
```

Reload nginx
```
sudo systemctl reload nginx
```

Disable nginx from startup boot
```
sudo systemctl disable nginx
```

Enable nginx to start on startup boot
```
sudo systemctl enable nginx
```

### Install Nodejs/Npm

Update local package index
```
sudo apt update
```

Install nodejs package
```
sudo apt install nodejs
```

Install npm
```
sudo apt install npm
```

### Create simple nodejs application
Create simple nodejs application which run on `port :3000`
```
mkdir /var/www/node-simple-app
cd /var/www/node-simple-app
nano app.js
```

Put following code
```javascript
const express = require('express')
const app = express()
const port = 3000

app.get('/', (req, res) => res.send('Hello World!'))

app.listen(port, () => console.log(`Example app listening on port ${port}!`))
```

Save and closed. Run
```
npm init
npm install --save express
```

Run application. Make sure to always run this application to make `port :3000` always open.
```
node app.js
```

### Create simple PHP application
Create simple nodejs application which run on `port :8080`
```
mkdir /var/www/php-simple-app
cd /var/www/php-simple-app
nano index.php
```

Put following code
```php
<?php
phpinfo();
?>
```

Save and closed. Disable default vhost
```
cd /etc/apache2/sites-available
sudo a2dissite 000-default.conf
```

Create new vhost
```
sudo cp 000-default.conf php-simple-app.conf
nano php-simple-app.conf
```

Put following code 
```
<VirtualHost *:8080>
        ServerName 000.00.000.000 # Remote ip address

        ServerAdmin boyblg@vk.com
        DocumentRoot /var/www/php-simple-app

        ErrorLog ${APACHE_LOG_DIR}/error.log
        CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
```

Enable new vhost
```
sudo a2ensite php-simple-app.conf
```

Reload apache2
```
sudo systemctl reload apache2
```

### Reverse Proxy from nginx to PHP site
Pass traffic from `port :80` nginx into `port :8080` of PHP app. Create nginx server block
```
nano /etc/nginx/sites-available/php-simple-app.conf
```

Put following basic content
```
server {
        listen 80;
        listen [::]:80;

        server_name php-test.example.com;

        location / {

                proxy_pass_header Authorization;
                proxy_pass http://000.00.000.000:8080; # pass traffic to apache with listen to port 8080
                proxy_set_header Host $host;
                proxy_set_header X-Real-IP $remote_addr;
                proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
                proxy_set_header X-Forwarded-Proto https;
                proxy_http_version 1.1;
                proxy_set_header Connection "";
                proxy_buffering off;
                client_max_body_size 0;
                proxy_read_timeout 36000s;
                proxy_redirect off;
        }
}
```

### Reverse Proxy from nginx to Node site
Pass traffic from `port :80` nginx into `port :3000` of nodejs. Create nginx server block
```
nano /etc/nginx/sites-available/node-simple-app.conf
```

Put following basic content
```
server {
        listen 80;
        listen [::]:80;

        server_name node-test.example.com;

        location / {

                proxy_pass_header Authorization;
                proxy_pass http://000.00.000.000:3000; # pass traffic to apache with listen to port 3000
                proxy_set_header Host $host;
                proxy_set_header X-Real-IP $remote_addr;
                proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
                proxy_http_version 1.1;
                proxy_set_header Connection "";
                proxy_buffering off;
                client_max_body_size 0;
                proxy_read_timeout 36000s;
                proxy_redirect off;
        }
}
```

Disable default server block
```
sudo mv /etc/nginx/sites-available/default /etc/nginx/sites-available/default.disabled 
```

Delete default server block
```
sudo rm /etc/nginx/sites-enabled/default
```

Enable PHP apps nginx server block
```
sudo ln -s /etc/nginx/sites-available/php-simple-app.conf /etc/nginx/sites-enabled/php-simple-app.conf 
```

Enable Node apps nginx server block
```
sudo ln -s /etc/nginx/sites-available/node-simple-app.conf /etc/nginx/sites-enabled/node-simple-app.conf 
```

Test nginx config
```
nginx -t
```

Restart nginx
```
sudo systemctl restart nginx
```

### Setting up SSL for both apps
Since we're using nginx as a front web server, certbot must be installed by using nginx configuration. Add certbot PPA

```
sudo apt-get update
sudo apt-get install software-properties-common
sudo add-apt-repository universe
sudo add-apt-repository ppa:certbot/certbot
sudo apt-get update
```

Install certbot
```
sudo apt-get install certbot python3-certbot-nginx
```

Run automate ssl installation by using this command
```
sudo certbot --nginx
```

Installing mariadb in Debian 11 or Ubuntu 20.04
```
apt install mariadb-server
```

Run the mysql initial configuration script and set the password for root. Everything else can be left by default.
```
mysql_secure_installation
```

A series of dialogs opens where you need to make some changes to the security settings of the MariaDB installation. We change the password of the root user for the current database, then delete anonymous users, disable the ability to connect root remotely, delete the test user and the database. all suggestions of the master.

The mysql/mariadb configuration files in Debian are located in the /etc/mysql/ directory. For normal operation, the default settings are sufficient. But if you decide to change them, don't forget to restart the database service.
```
service mariadb restart
```

## Installing phpmyadmin
In order to install phpmyadmin on our web server, it is enough to simply unpack the panel sources into a directory with a virtual host. Creating a folder structure.
```
mkdir /var/www/php-simple-app/pma
```
Go to the website https://www.phpmyadmin.net and copy the link to the latest version of the panel. Then we load it through the console
```
cd ~
wget https://files.phpmyadmin.net/phpMyAdmin/5.2.0/phpMyAdmin-5.2.0-all-languages.tar.gz
```

Unpack the sources into the virtual host directory
```
tar xvzf phpMyAdmin-5.2.0-all-languages.tar.gz
cp -R phpMyAdmin-5.2.0-all-languages/* /var/www/php-simple-app/pma/
chown -R root. /var/www/php-simple-app/pma/
```

##Setting up phpmyadmin

Phpmyadmin is ready to work immediately after installation, additional settings are not required. We close access to the panel by means of the Web server. To use the panel, you will need not only to know the mysql account name, but also the user and password to access the panel directly.

We will use a standard tool to restrict access to the directory using .htaccess. Let's create such a file in the folder with phpmyadmin scripts:
```
nano /var/www/php-simple-app/pma/.htaccess
```
```

AuthName "Enter Password"
AuthType Basic
Require valid-user
AuthUserFile "/var/www/php-simple-app/pma/.htpasswd"
```
Now let's create a file with authorization data:
```
htpasswd -bc /var/www/php-simple-app/pma/.htpasswd user password
```

Where user is the username and password is the password.

In order for authorization to work, it is necessary to add the AllowOverride parameter before </VirtualHost> in the virtual hosting configuration file in the Directory section
```
nano /etc/apache2/sites-available/php-simple-app.conf
```
```
<Directory /home/yuri/www/test.hserv.su/pma/>
  AddDefaultCharset UTF-8
  Require all granted
  AllowOverride All
</Directory>
```

Restarting Apache.
```
service apache2 restart
```
Checking the setting. When accessing the address of the web panel, a window with authorization should pop up.


Done

## Test
When navigate to `https://php-test.example.com`, nginx will redirect traffic to `https://<localhost|server-ip>:8080` which serve the PHP site.

When navigate to `https://php-test.example.com/pma`, phpmyadmin should open.

If you get error #1698 - Access denied for user ‘root’@’localhost’, then use this:

```
sudo mysql
```

```
use mysql;
ALTER USER 'root'@'localhost' IDENTIFIED BY 'PASSWORD';
exit
```
Where is "PASSWORD", the password to log in to phpmyadmin

When navigate to `https://node-test.example.com`, nginx will redirect traffic to `https://<localhost|server-ip>:3000` which serve the Node JS site.
