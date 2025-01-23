# Install MariaDB
ERPNext is built to naively run on MariaDB. Lets go ahead and set that up now.

```console
sudo apt update && sudo apt upgrade -y
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

# Edit MYSQL default config file
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