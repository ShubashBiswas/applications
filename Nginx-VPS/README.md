# # My-Applications
------------------------------------
## VPS Applications
1. Ubuntu - 24.04 'Noble Numbat' (LTS)
2. Nginx - latest (1.27.2)
------------------------------------
## Database Applications
1. MySQL - 8.4 (LTS)
2. MariaDB - 11.4 (LTS)
3. PostgreSQL - latest (16.4)
4. redis - latest (7.4)
5. phpMyAdmin - latest (5.2.1)
------------------------------------
## Application Language
1. PHP - 8.3
------------------------------------
## ERP Applications
1. ERPNext - latest (15.38.0)
------------------------------------
## CMS Applications
1. WordPress - latest (6.6.2)
2. OJS - latest(3.4.0-7)
------------------------------------
## Newsletter Applications
1. Listmonk - latest (3.0.0)
------------------------------------
## Email Server
1. Stalwart Mail Server - latest (0.10.2)
2. docker-mailserver (without gui) - latest (v14.0.0) ([docker-mailserver](https://github.com/docker-mailserver/docker-mailserver))
3. mailcow - latest (2024-08a) - very resource intensive - not recommended
## Email client
1. SnappyMail


Step 1 — Installing Nginx
1. $ sudo apt update
2. $ sudo apt install nginx
3. sudo ufw allow 'Nginx HTTP' - optinal
4. systemctl status nginx

Step 2 — Configuring your Server Block
1. sudo nano /etc/nginx/sites-available/your_domain
Insert the following into your new file, making sure to replace your_domain and app_server_address. If you do not have an application server to test with, default to using http://127.0.0.1:8000 for the optional Gunicorn server setup in Step 3:

/etc/nginx/sites-available/your_domain

server {
    listen 80;
    listen [::]:80;

    server_name your_domain www.your_domain;
        
    location / {
        proxy_pass app_server_address;
        include proxy_params;
    }
}

/etc/nginx/proxy_params

proxy_set_header Host $http_host;
proxy_set_header X-Real-IP $remote_addr;
proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
proxy_set_header X-Forwarded-Proto $scheme;

Next, enable this configuration file by creating a link from it to the sites-enabled directory that Nginx reads at startup:

$ sudo ln -s /etc/nginx/sites-available/your_domain /etc/nginx/sites-enabled/
$ sudo nginx -t
$ sudo systemctl restart nginx

Step 3 — Testing your Reverse Proxy with Gunicorn (Optional)
$ sudo apt update
$ sudo apt install gunicorn
$ nano test.py
def app(environ, start_response):
    start_response("200 OK", [])
    return iter([b"Hello, World!"])

$ gunicorn --workers=2 test:app