## Install instructions
This is a guide for running ERPNext to Ubuntu with Nginx

## Server Settings
1. Update and Upgrade Packages

```console
sudo apt-get update -y
sudo apt-get upgrade -y
```

# Create a new user – (bench user)
In Linux, the root user processes escalated privileges to perform any tasks within the system. This is why it is not advisable to use this user on a daily basis. We will create a user that we can use, and this will be the user we will also use as the Frappe Bench User.

```console
> sudo adduser [frappe-user]
usermod -aG sudo [frappe-user]
su [frappe-user]
cd /home/[frappe-user]
```

Ensure you have replaced [frappe-user] with your username. eg. sudo adduser frappe

# Install Required Packages
A software like ERPNext, which is built on Frappe Framework, requires a number of packages in order to run smoothly. These are the packages we will be installing in this step.

Install GIT
```console
sudo apt-get install git -y
```
Install Python Dependencies

Installing ERPNext version 15 on Ubuntu 24.04 requires Python version 3.12+. This is what we will install in this step.

Python-dev
```console
sudo apt-get install python3-dev -y
```
Setuptools and pip
```console
sudo apt-get install python3-setuptools python3-pip -y
```

Virtualenv
```console
sudo apt install python3.12-venv -y
```

Install MariaDB
ERPNext is built to naively run on MariaDB. Lets go ahead and set that up now.

```console
sudo apt-get install software-properties-common -y
sudo apt install mariadb-server -y
sudo mysql_secure_installation
```

When you run this command, the server will show the following prompts. Please follow the steps as shown below to complete the setup correctly.

Enter current password for root: (Enter your SSH root user password)

Switch to unix_socket authentication [Y/n]: Y

Change the root password? [Y/n]: Y

It will ask you to set new MySQL root password at this step. This can be different from the SSH root user password.

Remove anonymous users? [Y/n] Y

Disallow root login remotely? [Y/n]: N

This is set as N because we might want to access the database from a remote server for using business analytics software like Metabase / PowerBI / Tableau, etc.

Remove test database and access to it? [Y/n]: Y

Reload privilege tables now? [Y/n]: Y

Edit MYSQL default config file
```console
sudo nano /etc/mysql/my.cnf
```
Add the following block of code to the end of the file, exactly as is:

```console
[mysqld]
character-set-client-handshake = FALSE
character-set-server = utf8mb4
collation-server = utf8mb4_unicode_ci

[mysql]
default-character-set = utf8mb4
```

Restart the MYSQL Server
sudo service mysql restart

Install Redis Server
```console
sudo apt-get install redis-server -y

```
# Install CURL, Node, NPM and Yarn
Install CURL
```console
sudo apt install curl
```
Install Node
```console
curl https://raw.githubusercontent.com/creationix/nvm/master/install.sh | bash
source ~/.profile
nvm install 22
```

Install NPM
```console
sudo apt-get install npm -y
```
Install Yarn
```console
sudo npm install -g yarn -y
```
Install wkhtmltopdf
```console
sudo apt-get install xvfb libfontconfig wkhtmltopdf -y
```

Install Frappe Bench

In this step, we need to supply the – H and –break-system-packages flags. The flags do the following:
-H: Sets the HOME environment variable to the home directory of the target user. This ensures that the package installation happens in the correct directory and avoids permission issues.

--break-system-packages: Overrides the default behavior of pip to avoid breaking system packages. This flag is used to indicate that you are aware that this installation might cause conflicts with the system package manager, but you want to proceed anyway.

```console
sudo -H pip3 install frappe-bench --break-system-packages
```
Install ansible
```console
sudo -H pip3 install ansible --break-system-packages
```
Initialize Frappe Bench
```console
bench init frappe-bench --frappe-branch version-15
```
Switch directories into the Frappe Bench directory
```console
cd frappe-bench
```
Change user directory permissions
This will give the bench user execution permission to the home directory.
```console
chmod -R o+rx /home/[frappe-user]
```
Create a New Site

A site is a requirement in ERPNext, Frappe and all the other apps we will be needing to install. We will create the site in this step.
```console
bench new-site [site-name]
```
Install ERPNext and other Apps

Download all the apps we want to install
The first app we will download is the payments app. This app is required when setting up ERPNext.
```console
bench get-app payments
```
Next, we will download ERPNext app
```console
bench get-app --branch version-15 erpnext
```
Download any other app you may be interested in in a similar manner. 
```console
bench get-app webshop
```
Install all the apps on our site
```console
bench --site [site-name] install-app erpnext
```
Install all the other apps you downloaded in the same way.
```console
bench --site [site-name] install-app webshop
```
We have successfully setup ERPNext version 15 on ubuntu 24.04. You can start the server by running the below command:
```console
bench start
```
If you didn’t have any other ERPNext instance running on the same server, ERPNext will get started on port 8000. If you visit [YOUR SERVER IP:8000], you should be able to see ERPNext version 15 running.

Please note that instances which are running on develop mode, like the one we just setup, will not get started when you restart your server. You will need to run the bench start command every time the server restarts.

In the below steps, we will learn how to deploy the production mode.

Setting ERPNext for Production
Enable Scheduler
```console
bench --site [site-name] enable-scheduler
```
Disable maintenance mode
```console
bench --site [site-name] set-maintenance-mode off
```
Setup production config
```console
sudo bench setup production [frappe-user]
```
Setup NGINX to apply the changes
```console
bench setup nginx
```
Restart Supervisor and Launch Production Mode
```console
sudo supervisorctl restart all
sudo bench setup production [frappe-user]
```

If you are prompted to save the new/existing config file, respond with a Y.

When this completes doing the settings, your instance is now in production mode and can be accessed using your IP, without needing to use the port.

How to implement multi-tenancy in ERPNext and the Frappe Framework.

Steps needed
```console
bench config dns_multitenant on
bench new-site first.site
bench setup nginx
sudo service nginx reload
bench --site first.site install-app erpnext
```

After this is successful, navigate to /etc/hosts and make the following changes.

Please take into account the name of your sites. You will need to add a route for every site you add.

Add routes:
```markdown
127.0.0.1 first.site
127.0.0.1 second.site
```

Steps to Install SSL
Install snapd on your machine

```console
sudo apt install snapd

```
Update snapd
```console
sudo snap install core;
sudo snap refresh core
```

Remove existing installations of certbot
```markdown
sudo apt-get remove certbot
```
Install certbot
```console
sudo snap install --classic certbot
sudo ln -s /snap/bin/certbot /usr/bin/certbot
```

For one-step automatic ssl installation
```console
sudo certbot --nginx
```

If you prefer manual installation,. do the following:
```console
sudo certbot certonly --nginx
```

Certbot packages on your system come with a cron job or systemd timer that will renew your certificates automatically before they expire. So there's no need for any more steps. If necessary, you can test automatic renewal for your certificates by running this command:

```console
sudo certbot renew --dry-run
```

--------------------------------
Share frappe site over local network on Ubuntu 24.04
1. run [ip addr | grep inet] note down wlan0 or eth0 network IP.
2. enable ufw
3. allow 80 and 8000

I wish you all the best.

# For more frappe apps follow below commands

# Core Apps:
	frappe
	erpnext
	payments
	webshop (eCommerce)
-----------------------------------------
# Additional-Apps:
	> hrms
	insights
	lms
	lending
	builder
	helpdesk
	healthcare	
	crm
	print_designer
	wiki
	gameplan
	ury (https://github.com/ury-erp/ury/blob/develop/INSTALLATION.md)
	drive
	Front Desk https://github.com/core-initiative/front-desk (Hotel Front Office)
	ecommerce_integrations
	erpnext-shipping
	erpnext_price_estimation
	non_profit
	education
	changemakers
	restaurant_management (https://github.com/quantumbitcore/erpnext-restaurant.git)
	agriculture
	biometric-attendance-sync-tool
	waba_integration (WhatsApp Business)
	chat
	hospitality ( manage hotels & restaurants)
	bench_manager
	nextcloud-integration
-----------------------------------------
# Standalone Application: books
-----------------------------------------