# Steps
## Step 1 – Installing Nginx
```console
sudo apt install nginx
```

## Step 2 – Checking your Web Server
```console
systemctl status nginx
```

## Step 3 – Managing the Nginx Process
```console
sudo systemctl start nginx
sudo systemctl restart nginx
sudo systemctl reload nginx
sudo systemctl disable nginx
sudo systemctl enable nginx
```

## Step 4 – Setting Up Server Blocks
```console
sudo mkdir -p /var/www/your_domain/html
sudo chown -R $USER:$USER /var/www/your_domain/html
sudo chmod -R 755 /var/www/your_domain
sudo nano /var/www/your_domain/html/index.html
```

```htm
<html>
    <head>
        <title>Welcome to your_domain!</title>
    </head>
    <body>
        <h1>Success!  The your_domain server block is working!</h1>
    </body>
</html>
```

```console
sudo nano /etc/nginx/sites-available/your_domain
```

```console
server {
        listen 80;
        listen [::]:80;

        root /var/www/your_domain/html;
        index index.html index.htm index.nginx-debian.html;

        server_name your_domain www.your_domain;

        location / {
                try_files $uri $uri/ =404;
        }
}
```

```console
sudo ln -s /etc/nginx/sites-available/your_domain /etc/nginx/sites-enabled/
```

```console
sudo nano /etc/nginx/nginx.conf
```

```console
http {
    ...
    server_names_hash_bucket_size 64;
    ...
}
```

```console
sudo nginx -t
sudo systemctl restart nginx
```