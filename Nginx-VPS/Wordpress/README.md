## Steps
# Step 1 – Installing Nginx
# Step 2 – Install MariaDB Server
```console
sudo apt install mariadb-server
sudo service mysql stop
sudo service mysql start
sudo mysql_secure_installation
```

# Step 3 – Managing the Nginx Process
```console
sudo apt install php-fpm php-common php-mysql php-gmp php-curl php-intl php-mbstring php-xmlrpc php-gd php-xml php-cli php-zip

sudo nano /etc/php/8.3/fpm/php.ini
```

```markdown
file_uploads = On
allow_url_fopen = On
short_open_tag = On
memory_limit = 256M
cgi.fix_pathinfo = 0
upload_max_filesize = 100M
max_execution_time = 360
date.timezone = America/Chicago
```

```markdown
sudo service php8.3-fpm stop
sudo service php8.3-fpm start
```

# Step 4 – Create WordPress Database
```console
sudo mysql -u root -p
```
```markdown
CREATE DATABASE wpdb;
CREATE USER 'wpdbuser'@'localhost' IDENTIFIED BY 'new_password_here';
GRANT ALL ON wpdb.* TO 'wpdbuser'@'localhost' WITH GRANT OPTION;
FLUSH PRIVILEGES;
EXIT;
```

# Step 5 – Create WordPress
```markdown
cd /tmp
wget https://wordpress.org/latest.tar.gz
tar -xvzf latest.tar.gz
sudo mv wordpress /var/www/wordpress
```

```console
sudo chown -R www-data:www-data /var/www/wordpress/
sudo chmod -R 755 /var/www/wordpress/
```

# Step 6 – Configure the Nginx Server block.
```console
sudo nano /etc/nginx/sites-available/wordpress
```
```console
server {
    listen 80;
    listen [::]:80;
    root /var/www/wordpress;
    index  index.php index.html index.htm;
    server_name  example.com www.example.com;

    client_max_body_size 100M;
    autoindex off;
    
    location / {
        try_files $uri $uri/ /index.php?$args;
    }

    location ~ \.php$ {
         include snippets/fastcgi-php.conf;
         fastcgi_pass unix:/var/run/php/php7.4-fpm.sock;
         fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
         include fastcgi_params;
    }
}
```
```console
sudo ln -s /etc/nginx/sites-available/wordpress /etc/nginx/sites-enabled/
sudo service nginx restart
```
Visit example.com for wordpress web installation 