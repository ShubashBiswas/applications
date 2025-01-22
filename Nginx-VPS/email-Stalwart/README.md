## Install instructions
Setting up a **Stalwart Mail Server** on an Ubuntu VPS with Nginx involves several steps. Stalwart Mail is a lightweight and efficient email server that supports IMAP, SMTP, and JMAP protocols. Below is a detailed guide to help you install and configure it on your Ubuntu VPS.

---

### **Prerequisites**
1. **Ubuntu VPS** with root or sudo privileges.
2. **Domain Name** pointed to your VPS's IP.
3. Basic familiarity with Linux command-line tools.

---

### **Step 1: Update the System**
```bash
sudo apt update && sudo apt upgrade -y
```

### **Step 2: install Stalwart Mail Server on Linux**
To install Stalwart Mail Server on Linux or MacOS, execute the following command in your terminal:

```bash
$ curl --proto '=https' --tlsv1.2 -sSf https://get.stalw.art/install.sh -o install.sh
```

```htm
✅ Configuration file written to /opt/stalwart-mail/etc/config.toml
🔑 Your administrator account is 'admin' with password 'password'.
Created symlink /etc/systemd/system/multi-user.target.wants/stalwart-mail.service → /etc/systemd/system/stalwart-mail.service.
🎉 Installation complete! Continue the setup at http://IP.:8080/login
```

### **Step 6: Configure Nginx for Stalwart Mail**
Edit the Nginx configuration to reverse-proxy the JMAP service:
```bash
sudo nano /etc/nginx/sites-available/stalwart-mail
```

Add the following:
```nginx
server {
    listen 80;
    server_name mail.yourdomain.com;

    location / {
        proxy_pass http://127.0.0.1:8000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }

    location /acme-challenge/ {
        root /var/www/html;
    }
}

server {
    listen 443 ssl;
    server_name mail.yourdomain.com;

    ssl_certificate /etc/letsencrypt/live/mail.yourdomain.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/mail.yourdomain.com/privkey.pem;

    location / {
        proxy_pass http://127.0.0.1:8000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
}
```

Enable the configuration:
```bash
sudo ln -s /etc/nginx/sites-available/stalwart-mail /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl reload nginx
```

### **Step 3: Configure SSL**
1. Set up SSL with Let's Encrypt (recommended for security):
   ```bash
   sudo apt install -y certbot python3-certbot-nginx
   sudo certbot --nginx -d mail.yourdomain.com
   ```
   Replace `mail.yourdomain.com` with your domain name.
2. Verify SSL auto-renewal:
   ```bash
   sudo certbot renew --dry-run
   ```