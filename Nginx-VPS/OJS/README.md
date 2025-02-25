## Steps
# Step 1 – Install Nginx
# Step 2 – Install PHP with php-mbstring, php-xml, and php-intl
# Step 3 – Install MariaDB
# Step 3 – Create OJS DataBase

```console
mysql -u root -p
CREATE DATABASE ojs_db DEFAULT CHARACTER SET utf8 COLLATE utf8_unicode_520_ci;
CREATE USER 'ojs_user'@'localhost' IDENTIFIED BY 'ojs_password';
GRANT ALL ON ojs_db.* TO 'ojs_user'@'localhost';
FLUSH PRIVILEGES;
EXIT;
```

# Step 4 – Configuring Nginx
```console
sudo nano /etc/nginx/sites-available/ojs
```
```markdown
server {
    listen 80;
    listen [::]:80;

    root /var/www/ojs;
    index index.php;

    server_name server_domain_or_IP;

    location = /favicon.ico { log_not_found off; access_log off; }
    location = /robots.txt { log_not_found off; access_log off; allow all; }
    location ~* \.(css|gif|ico|jpeg|jpg|js|png)$ {
        expires max;
        log_not_found off;
    }
    location / {
    	try_files $uri $uri/ /index.php?$args;
    }

location ~ ^(.+\.php)(.*)$ {
        set $path_info $fastcgi_path_info;
        fastcgi_split_path_info ^(.+\.php)(.*)$;
        fastcgi_param PATH_INFO $path_info;
        fastcgi_param PATH_TRANSLATED $document_root$path_info;

        if (!-f $document_root$fastcgi_script_name) {
            return 404;
        }

        include snippets/fastcgi-php.conf;
        fastcgi_pass unix:/var/run/php/php8.3-fpm.sock;
    }
}
```

```console
sudo ln -s /etc/nginx/sites-available/ojs /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl restart nginx
```

# Step 4 – Installing Open Journal System
```markdown
cd /tmp
wget https://pkp.sfu.ca/ojs/download/ojs-3.4.0-8.tar.gz
tar xzvf ojs-3.4.0-8.tar.gz 
sudo cp -a /tmp/ojs-3.4.0-8/. /var/www/ojs
```

```console
sudo chown -R www-data:www-data /var/www/ojs
sudo mkdir /var/www/ojs-files
sudo chown -R www-data:www-data /var/www/ojs-files
```
Let's visit the following link on a browser.
[http://server_domain_or_IP/](https://)

# Step 5 – VS Code Permissions.
Change File Ownership (Recommended for Development)
```console
sudo usermod -aG www-data your-username
sudo chown -R www-data:www-data /var/www/
sudo chmod -R g+rw /var/www
```
