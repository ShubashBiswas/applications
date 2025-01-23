# Install Required and Recommended PHP Modules.
Run the following command to install PHP modules required or recommended by phpMyAdmin.

```console
sudo apt install -y php-imagick php-phpseclib php-php-gettext php8.3-common php8.3-mysql php8.3-fpm php8.3-gd php8.3-imap php8.3-curl php8.3-zip php8.3-xml php8.3-mbstring php8.3-bz2 php8.3-intl php8.3-gmp
```
## Configure PHP

```markdown
file_uploads = On
allow_url_fopen = On
short_open_tag = On
memory_limit = 256M
cgi.fix_pathinfo = 0
upload_max_filesize = 100M
max_execution_time = 360
date.timezone = America/Chicago
```

```markdown
sudo service php8.3-fpm stop
sudo service php8.3-fpm start
```