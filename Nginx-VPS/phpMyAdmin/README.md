## Step 1: Download phpMyAdmin on Ubuntu 24.04 Server
phpMyAdmin is included in the default Ubuntu 24.04 software repository, but speaking from my experience, it’s better to install the latest version using the upstream package. Run the following command to download it.

```console
wget https://www.phpmyadmin.net/downloads/phpMyAdmin-latest-all-languages.zip
```
Then extract it.

```console
sudo apt install unzip
```

```console
unzip phpMyAdmin-latest-all-languages.zip
```Move phpMyadmin to /var/www/ directory.

```console
sudo mkdir -p /var/www/
```

```console
sudo mv phpMyAdmin-*-all-languages /var/www/phpmyadmin
```
Then make the web server user (www-data) as the owner of this directory.

```console
sudo chown -R www-data:www-data /var/www/phpmyadmin
```
## Step 2: Create a MariaDB Database and User for phpMyAdmin
Log in to MariaDB console if .

```console
sudo mysql -u root
```
Create a new database for phpMyAdmin using the following SQL command. This tutorial names it phpmyadmin, you can use whatever name you like for the database.

```console
CREATE DATABASE phpmyadmin DEFAULT CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
```
The following SQL command will create the phpmyadmin database user and set a password, and at the same time grant all permission of the new database to the new user so later on phpMyAdmin can write to the database. Replace red texts with your preferred password.

```console
GRANT ALL ON phpmyadmin.* TO 'phpmyadmin'@'localhost' IDENTIFIED BY 'your_preferred_password';
```
Flush the privileges table and exit the MariaDB console.

```console
FLUSH PRIVILEGES;
EXIT;
```
## Step 3: Install Required and Recommended PHP Modules.
Run the following command to install PHP modules required or recommended by phpMyAdmin.

```console
sudo apt install -y php-imagick php-phpseclib php-php-gettext php8.3-common php8.3-mysql php8.3-fpm php8.3-gd php8.3-imap php8.3-curl php8.3-zip php8.3-xml php8.3-mbstring php8.3-bz2 php8.3-intl php8.3-gmp
```
## Step 4: Create Nginx Server Block for phpMyAdmin
To be able to access the phpMyAdmin web interface, we need to create a Nginx server block by running the following command.

sudo nano /etc/nginx/conf.d/phpmyadmin.conf
We will configure it so that we can access phpMyAdmin via a sub-domain. Paste the following text into the file. Replace pma.example.com with your actual sub-domain and don’t forget to create DNS A record for it.

```htm
server {
  listen 80;
  listen [::]:80;
  server_name pma.example.com;
  root /var/www/phpmyadmin/;
  index index.php index.html index.htm index.nginx-debian.html;

  access_log /var/log/nginx/phpmyadmin_access.log;
  error_log /var/log/nginx/phpmyadmin_error.log;

  location / {
    try_files $uri $uri/ /index.php;
  }

  location ~ \.php$ {
    fastcgi_pass unix:/run/php/php8.3-fpm.sock;
    fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
    include fastcgi_params;
    include snippets/fastcgi-php.conf;
  }

  location ~ /\.ht {
    deny all;
  }
}
```

Your phpMyAdmin files are in /var/www/phpmyadmin/ directory. Save and close the file. Then test Nginx configurations.

```console
sudo nginx -t
```
If the test is successful, reload Nginx for the changes to take effect.

```console
sudo systemctl reload nginx
```
Now you should be able to access phpMyAdmin web interface via

```console
pma.example.com
```
how-to-install-phpmyadmin-in-ubuntu

## Step 5: Installing TLS Certificate
To secure the phpMyadmin web interface, we can install a free Let’s Encrypt TLS certificate. Install the Let’s Encrypt client from Ubuntu 24.04 software repository like below:

```console
sudo apt install certbot python3-certbot-nginx
```
Python3-certbot-nginx is the Nginx plugin for Certbot. Now run the following command to obtain and install TLS certificate.

```console
sudo certbot --nginx --agree-tos --redirect --hsts --staple-ocsp -d pma.example.com --email you@example.com
```
Where:

–nginx: Use the Nginx authenticator and installer
–agree-tos: Agree to Let’s Encrypt terms of service
–redirect: Enforce HTTPS by 301 redirect.
–hsts: Add the Strict-Transport-Security header to every HTTP response.
–staple-ocsp: Enables OCSP Stapling.
–must-staple: Adds the OCSP Must Staple extension to the certificate.
-d flag is followed by a list of domain names, separated by a comma. You can add up to 100 domain names.
–email: Email used for registration and recovery contact.
You will be asked if you want to receive emails from EFF(Electronic Frontier Foundation). After choosing Y or N, your TLS certificate will be automatically obtained and configured for you, which is indicated by the message below.

phpmyadmin-https-ubuntu-24.04

# Step 6: Create the phpMyAdmin Config File
Copy the example config file.

```console
sudo cp /var/www/phpmyadmin/config.sample.inc.php /var/www/phpmyadmin/config.inc.php
```
Now edit the new file.

```console
sudo nano /var/www/phpmyadmin/config.inc.php
```
Find the following line.

```htm
$cfg['blowfish_secret'] = ''; /* YOU MUST FILL IN THIS FOR COOKIE AUTH! */
```

We need to enter a 32-bytes long string of random bytes like below.

```htm
$cfg['blowfish_secret'] = 'eYU7tk3jmNLYDGYJ9h7AZE2BYXx4UHSQ'; /* YOU MUST FILL IN THIS FOR COOKIE AUTH! */
```
Save and close the file.

Step 7: Troubleshoot phpMyAdmin Login Error
If you log in with MariaDB root account, you may see the following error.

 #1698 - Access denied for user 'root '@'localhost'
and

mysqli_real_connect(): (HY000/1698): Access denied for user 'root '@'localhost'
If you log in with user phpmyadmin, you won’t see the above error. However, user phpmyadmin can only be used to administer the phpmyadmin database.

The cause of the error is that by default MariaDB root user is authenticated via the unix_socket plugin, instead of using the mysql_native_password plugin. To get around this issue, we can create another admin user and grant all privileges to the new admin user.

Log into the MariaDB server from the command line.

```console
sudo mariadb -u root
```
Create an admin user with password authentication.

```console
create user admin@localhost identified by 'your-chosen-password';
```
Grant all privileges on all databases.

```console
grant all privileges on *.* to admin@localhost with grant option;
```
Flush privileges and exit;

```console
flush privileges;
exit;
```
Now you can log into phpMyAdmin with the admin account and manage all databases.

Step 8: Restricting Access to the /setup Directory
Since the phpMyAdmin setup is finished, there’s no need to allow access to the /setup directory.

Edit Nginx configuration file.

```console
sudo nano /etc/nginx/conf.d/phpmyadmin.conf
```
Add the following code snippet to this file.

```htm
  location ~ ^/(doc|sql|setup)/ {
    deny all;
  }
```
Save and close the file. Then test Nginx configurations.

```console
sudo nginx -t
```
If the test is successful, reload Nginx for the changes to take effect.

```console
sudo systemctl reload nginx
```
TLS Certificate Auto-Renewal
To automatically renew Let’s Encrypt certificate, simply edit root user’s crontab file.

```console
sudo crontab -e
```
Then add the following line at the bottom.

```console
@daily certbot renew --quiet && systemctl reload nginx
```
Reloading Nginx is needed for it to pick up the new certificate to clients.

Use Webmin to Manage MySQL/MariaDB Databases
Webmin is a general Linux server admin panel. It also allows you to manage MySQL/MariaDB databases from a graphical user interface, although it’s not as feature-rich as phpMyAdmin when it comes to database management.

Create databases
Create database users
Grant permissions to database users.
How to Install Webmin on Ubuntu Server

Wrapping Up
I hope this tutorial helped you install phpMyAdmin with Nginx on Ubuntu 24.04 LTS.