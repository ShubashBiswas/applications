## Steps
# Step 1 – Install Nginx
# Step 2 – Install MariaDB Server
# Step 3 – Install PHP
# Step 4 – Create WordPress Database
```console
sudo mysql -u root -p
```
```markdown
CREATE DATABASE wpdb;
CREATE USER 'wpdbuser'@'localhost' IDENTIFIED BY 'wpdb_password_here';
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
    server_name  example.com;
    return 301 https://$host$request_uri;
    root /var/www/wordpress;
    index  index.php index.html index.htm;
    
    client_max_body_size 100M;
    autoindex off;
    
    location / {
        try_files $uri $uri/ /index.php?$args;
    }

    location ~ \.php$ {
        include snippets/fastcgi-php.conf;
        fastcgi_pass unix:/var/run/php/php8.3-fpm.sock;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        include fastcgi_params;
        try_files $uri =404;  # Prevent direct access to non-existing PHP files
    }
    location /.well-known/acme-challenge/ {
    allow all;
    root /var/www/html;
    }
}
```
```console
sudo ln -s /etc/nginx/sites-available/wordpress /etc/nginx/sites-enabled/
sudo service nginx restart
```
Visit example.com for WordPress web installation

# Step 7 – VS Code Permissions.
Change File Ownership (Recommended for Development)
```console
sudo usermod -aG www-data your-username
sudo chown -R www-data:www-data /var/www/
sudo chmod -R g+rw /var/www
```
Restart VS Code
