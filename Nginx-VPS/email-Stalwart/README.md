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

---

### **Step 2: Install Dependencies**
Stalwart Mail depends on certain libraries and tools:
```bash
sudo apt install -y build-essential libssl-dev libsqlite3-dev wget curl gnupg2 unzip
```

---

### **Step 3: Install and Configure Nginx**
1. Install Nginx:
   ```bash
   sudo apt install -y nginx
   ```
2. Set up SSL with Let's Encrypt (recommended for security):
   ```bash
   sudo apt install -y certbot python3-certbot-nginx
   sudo certbot --nginx -d mail.yourdomain.com
   ```
   Replace `mail.yourdomain.com` with your domain name.
3. Verify SSL auto-renewal:
   ```bash
   sudo certbot renew --dry-run
   ```

---

### **Step 4: Install Stalwart Mail**
1. Download the latest release of Stalwart Mail binaries:
   ```bash
   wget https://github.com/stalwartlabs/mail-server/releases/latest/download/stalwart-mail-linux-amd64.zip
   ```
2. Unzip and move the binaries to `/usr/local/bin`:
   ```bash
   unzip stalwart-mail-linux-amd64.zip
   sudo mv stalwart-mail /usr/local/bin/
   sudo chmod +x /usr/local/bin/stalwart-mail
   ```

---

### **Step 5: Configure Stalwart Mail**
1. Create the configuration directory:
   ```bash
   sudo mkdir -p /etc/stalwart-mail
   ```
2. Generate a basic configuration file:
   ```bash
   sudo nano /etc/stalwart-mail/stalwart.toml
   ```
   Example configuration:
   ```toml
   [server]
   hostname = "mail.yourdomain.com"
   data_dir = "/var/lib/stalwart-mail"

   [imap]
   listen = "0.0.0.0:143"

   [smtp]
   listen = "0.0.0.0:25"
   hostname = "mail.yourdomain.com"

   [jmap]
   listen = "0.0.0.0:8000"
   ```

3. Ensure the data directory exists:
   ```bash
   sudo mkdir -p /var/lib/stalwart-mail
   sudo chown -R $(whoami): /var/lib/stalwart-mail
   ```

---

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

---

### **Step 7: Start Stalwart Mail**
1. Create a systemd service file:
   ```bash
   sudo nano /etc/systemd/system/stalwart-mail.service
   ```
   Add the following:
   ```ini
   [Unit]
   Description=Stalwart Mail Server
   After=network.target

   [Service]
   ExecStart=/usr/local/bin/stalwart-mail --config /etc/stalwart-mail/stalwart.toml
   Restart=on-failure

   [Install]
   WantedBy=multi-user.target
   ```

2. Start and enable the service:
   ```bash
   sudo systemctl daemon-reload
   sudo systemctl start stalwart-mail
   sudo systemctl enable stalwart-mail
   ```

---

### **Step 8: Configure DNS**
Set up DNS records for your domain:
- **MX Record**: `mail.yourdomain.com` pointing to your VPS.
- **A Record**: `mail.yourdomain.com` pointing to your VPS IP.
- **SPF Record**: `v=spf1 mx -all`
- **DKIM and DMARC**: Generate keys and add the corresponding DNS records.

---

### **Step 9: Test Your Mail Server**
1. Use an email client (like Thunderbird) to connect to your IMAP/SMTP server.
2. Test sending and receiving emails.
3. Verify JMAP functionality by testing through a client supporting JMAP.

---

Let me know if you'd like help with DNS setup, SPF/DKIM/DMARC configuration, or testing!