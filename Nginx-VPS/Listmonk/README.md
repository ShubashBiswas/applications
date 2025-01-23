## Install instructions

Note - This app have a template use the template. If that dosenot work use below methods. 

# Steps
1. Update System and Install Dependencies

```console
sudo apt update && sudo apt upgrade -y
```
2. Install Postgres Database
```console
sudo apt install postgresql postgresql-contrib
sudo -u postgres psql
```

```console
create database listmonk;
create user listmonk with encrypted password 'listmonk0101';
grant all privileges on database listmonk to listmonk;
ALTER DATABASE listmonk OWNER TO listmonk;
GRANT USAGE, CREATE ON SCHEMA PUBLIC TO listmonk;
\q
```
3. Install listmonk
```console
cd /opt
mkdir listmonk
wget https://github.com/knadh/listmonk/releases/download/v4.1.0/listmonk_4.1.0_linux_amd64.tar.gz
tar -zxvf listmonk_4.1.0_linux_amd64.tar.gz
./listmonk --new-config
sudo nano config.toml
```
4. Edit the config
```htm
address = "localhost:9000"

# Database.
[db]
host = "localhost"
port = 5432
user = "listmonk"
password = "listmonk0101"

# Ensure that this database has been created in Postgres.
database = "listmonk"
```

5. Install the listmonk and run it
```console
./listmonk --install
./listmonk
```

6. Now we need to start listmonk, and obviously we want to use systemctl to keep it always running:

```console
sudo nano /etc/systemd/system/listmonk.service
```
I added the following to the newly create service:
```markdown
[Unit]
Description=listmonk service
After=network.target

[Service]
Type=simple
ExecStart=/opt/listmonk/listmonk
WorkingDirectory=/opt/listmonk
Restart=always
User=www-data

[Install]
WantedBy=multi-user.target
```

```console
sudo systemctl daemon-reload
sudo systemctl enable listmonk
sudo systemctl start listmonk
```
7. Configure nginx as Reverse Proxy
```console
sudo nano /etc/nginx/sites-available/listmonk
```
```markdown
server {
    listen 80;
    server_name yourdomain.com;

    location / {
        proxy_pass http://127.0.0.1:9000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```
```console
sudo ln -s /etc/nginx/sites-available/listmonk /etc/nginx/sites-enabled/
```

```console
sudo nginx -t
sudo systemctl reload nginx
```