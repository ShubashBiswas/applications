## Install instructions

Note - This app have a template use the template. If that dosenot work use below methods. 

#Steps
Binary¶
* Download the latest release and extract the listmonk binary. amd64 is the main one. It works for Intel and x86 CPUs.
* ./listmonk --new-config to generate config.toml. Then, edit the file.
* ./listmonk --install to install the tables in the Postgres DB.
* Run ./listmonk and visit http://localhost:9000.

1. sudo apt update
2. sudo apt install postgresql postgresql-contrib
3. sudo -u postgres psql
4. create database listmonk;
5. create user listmonk with encrypted password 'listmonk0101';
6. grant all privileges on database listmonk to listmonk;
ALTER DATABASE listmonk OWNER TO listmonk;
GRANT USAGE, CREATE ON SCHEMA PUBLIC TO listmonk;
7. \q
8. wget https://github.com/knadh/listmonk/releases/download/v3.0.0/listmonk_3.0.0_linux_amd64.tar.gz
9. tar -zxvf listmonk_3.0.0_linux_amd64.tar.gz
10. ./listmonk --new-config
11. sudo nano config.toml

12. Install the listmonk and run it
13. ./listmonk --install
14. ./listmonk

Now we need to start listmonk, and obviously we want to use systemctl to keep it always running:

1. nano /etc/systemd/system/listmonk.service
I added the following to the newly create service:
[Unit]
Description=ListMonk
Documentation=https://listmonk.app
After=system.slice multi-user.target postgresql.service

[Service]
User=sweetp
Type=simple

StandardOutput=syslog
StandardError=syslog
SyslogIdentifier=listmonk

WorkingDirectory=/var/www/vhosts/mydomain.com/newsletter
ExecStart=/var/www/vhosts/mydomain.com/newsletter/listmonk --static-dir=static

Restart=always
RestartSec=10

[Install]
WantedBy=multi-user.target

2. systemctl daemon-reload
3. systemctl enable listmonk
4. systemctl start listmonk

Additional Apache directives:
<Location />
	ProxyPass http://127.0.0.1:9000/
	ProxyPassReverse http://127.0.0.1:9000/

	Header always set Access-Control-Allow-Origin "https://mydomain.com"
	Header always set Access-Control-Max-Age "1000"
	Header always set Access-Control-Allow-Headers "X-Requested-With, Content-Type, Origin, Authorization, Accept, Client-Security-Token, Accept-Encoding"
	Header always set Access-Control-Allow-Methods "POST, GET, OPTIONS, DELETE, PUT"

	RewriteEngine On
	RewriteCond %{REQUEST_METHOD} OPTIONS
	RewriteRule ^(.*)$ $1 [R=200,L]
</Location>
