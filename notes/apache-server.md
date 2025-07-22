# WordPress Migration from Nginx to Apache - Setup Guide

## Overview
This document outlines the steps to migrate a WordPress site from Nginx to Apache web server on a Linux system.

## Prerequisites
- Ubuntu/Debian Linux server
- Root or sudo access
- Existing WordPress installation files in `/var/www/html`

## Installation Steps

### 1. Install Apache and Required PHP Modules
```bash
sudo apt update
sudo apt install apache2 libapache2-mod-php php-mysql php-curl php-gd php-mbstring php-xml php-xmlrpc php-soap php-intl php-zip
```

### 2. Configure Apache Virtual Host
Create a new configuration file:
```bash
sudo nano /etc/apache2/sites-available/wordpress.conf
```

Add the following configuration (adjust IP/domain as needed):
```apache
<VirtualHost *:80>
    ServerName yourdomain.com
    ServerAlias www.yourdomain.com
    DocumentRoot /var/www/html
    
    <Directory /var/www/html>
        Options FollowSymLinks
        AllowOverride All
        Require all granted
    </Directory>
    
    ErrorLog ${APACHE_LOG_DIR}/wordpress_error.log
    CustomLog ${APACHE_LOG_DIR}/wordpress_access.log combined
</VirtualHost>
```

for ip static vagrant
```

<VirtualHost 192.168.56.13:80>
    # Remove ServerName and ServerAlias since we're using IP directly
    DocumentRoot /var/www/html
    
    <Directory /var/www/html>
        Options FollowSymLinks
        AllowOverride All
        Require all granted
    </Directory>
    
    ErrorLog ${APACHE_LOG_DIR}/wordpress_error.log
    CustomLog ${APACHE_LOG_DIR}/wordpress_access.log combined
</VirtualHost>

```
### 3. Enable Site and Required Modules
```bash
sudo a2ensite wordpress.conf
sudo a2enmod rewrite
sudo systemctl restart apache2
```

### 4. Stop Nginx (if previously running)
```bash
sudo systemctl stop nginx
```

### 5. Configure .htaccess
Create or modify the .htaccess file:
```bash
sudo nano /var/www/html/.htaccess
```

Add standard WordPress rewrite rules:
```apache
# BEGIN WordPress
<IfModule mod_rewrite.c>
RewriteEngine On
RewriteBase /
RewriteRule ^index\.php$ - [L]
RewriteCond %{REQUEST_FILENAME} !-f
RewriteCond %{REQUEST_FILENAME} !-d
RewriteRule . /index.php [L]
</IfModule>
# END WordPress
```

### 6. Restart Apache
```bash
sudo systemctl restart apache2
```

## Verification
1. Access your WordPress site in a web browser
2. Check that all pages load correctly
3. Verify permalinks are working
4. Test admin dashboard functionality

## Troubleshooting
- If you encounter 404 errors, ensure `mod_rewrite` is enabled
- For permission issues, run:
  ```bash
  sudo chown -R www-data:www-data /var/www/html
  ```
- Check error logs:
  ```bash
  sudo tail -f /var/log/apache2/wordpress_error.log
  ```

## Maintenance
To completely remove Nginx (optional):
```bash
sudo systemctl disable nginx
sudo apt remove nginx nginx-common
```