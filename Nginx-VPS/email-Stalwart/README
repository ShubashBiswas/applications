## Install instructions
To install Stalwart Mail Server on Linux or MacOS, execute the following command in your terminal:
#Steps
1. $ curl --proto '=https' --tlsv1.2 -sSf https://get.stalw.art/install.sh -o install.sh
2. Then, execute the installation script as root. The default installation directory is /opt/stalwart-mail.
4. $ sudo sh install.sh
✅ Configuration file written to /opt/stalwart-mail/etc/config.toml
🔑 Your administrator account is 'admin' with password 'w95Yuiu36E'.
🎉 Installation complete! Continue the setup at http://yourserver.org:8080/login
5. Configure your hostname and domain
Next, make sure that the server hostname in Settings > Server > Network is correct. Then, add your main domain name in Management > Directory > Domains. After creating the domain, the interface will display the DNS records that you need to add to your domain's DNS settings.
6. Enable TLS
    *If you already have a TLS certificate for your server, you can upload it in the Settings > Server > TLS > Certificates section.
    *If you don't have a certificate, you can enable automatic TLS certificates from Let's Encrypt using ACME. To enable ACME, go to the Settings > Server > TLS > ACME Providers section and add Let's Encrypt as your ACME provider making sure that your server hostname is listed as one of the Subject Names. Stalwart supports the tls-alpn-01, dns-01 and http-01 challenges, if you are unsure which one to use, read the ACME challenge types documentation.
    *If you are running Stalwart behind a reverse proxy such as Traefik, Caddy, HAProxy or NGINX, you should skip this step and configure TLS in your reverse proxy instead.
7. Restart service
Once you have completed the setup instructions, restart Stalwart Mail server:

$ sudo systemctl restart stalwart-mail