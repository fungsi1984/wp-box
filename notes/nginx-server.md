# WordPress Development Workflow with Vagrant, Nginx, and PHP 8.1

## Table of Contents
1. [Environment Setup](#environment-setup)
2. [Software Installation](#software-installation)
3. [Database Configuration](#database-configuration)
4. [WordPress Installation](#wordpress-installation)
5. [Nginx Configuration](#nginx-configuration)
6. [PHP Configuration](#php-configuration)
7. [Troubleshooting](#troubleshooting)
8. [Daily Workflow](#daily-workflow)

## Environment Setup

First, initialize your Vagrant environment:

```bash
mkdir wordpress-project
cd wordpress-project
vagrant init ubuntu/focal64
```

Edit your Vagrantfile to include a private network IP:
```ruby
config.vm.network "private_network", ip: "192.168.56.13"
```

Start the Vagrant environment:
```bash
vagrant up
vagrant ssh
```

## Software Installation

Once inside the Vagrant VM, install required packages:

```bash
sudo apt update
sudo apt install -y nginx mysql-server php8.1-fpm php8.1-mysql
sudo apt install -y php-curl php-gd php-mbstring php-xml php-xmlrpc php-soap php-intl php-zip
```

## Database Configuration

Set up the MySQL database for WordPress:

```bash
sudo mysql -e "CREATE DATABASE wordpress DEFAULT CHARACTER SET utf8 COLLATE utf8_unicode_ci;"
sudo mysql -e "CREATE USER 'wordpressuser'@'localhost' IDENTIFIED BY 'password';"
sudo mysql -e "GRANT ALL PRIVILEGES ON wordpress.* TO 'wordpressuser'@'localhost';"
sudo mysql -e "FLUSH PRIVILEGES;"
```

## WordPress Installation

Download and configure WordPress:

```bash
sudo mkdir -p /var/www/wordpress
cd /var/www/wordpress
sudo wget https://wordpress.org/latest.tar.gz
sudo tar xzvf latest.tar.gz --strip-components=1
sudo rm latest.tar.gz
sudo chown -R www-data:www-data /var/www/wordpress
```

## Nginx Configuration

Create and enable the Nginx site configuration:

```bash
sudo nano /etc/nginx/sites-available/wordpress
```

Paste this configuration:
```nginx
server {
    listen 80;
    server_name _;
    root /var/www/wordpress;
    index index.php index.html index.htm;

    location / {
        try_files $uri $uri/ /index.php?$args;
    }

    location ~ \.php$ {
        include snippets/fastcgi-php.conf;
        fastcgi_pass unix:/var/run/php/php8.1-fpm.sock;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        include fastcgi_params;
        fastcgi_split_path_info ^(.+\.php)(/.+)$;
    }

    location ~ /\.ht {
        deny all;
    }
}
```

```
somehow this is works
server {
    listen 80;
    server_name _;
    root /var/www/html;
    index index.php index.html index.htm;

    location / {
        try_files $uri $uri/ /index.php?$args;
    }

    location ~ \.php$ {
        include snippets/fastcgi-php.conf;
        fastcgi_pass unix:/var/run/php/php8.1-fpm.sock;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        include fastcgi_params;

        # Add these for better security
        fastcgi_split_path_info ^(.+\.php)(/.+)$;
        # fastcgi_index index.php;
    }

    location ~ /\.ht {
        deny all;
    }
}

```

Enable the site and restart Nginx:
```bash
sudo ln -s /etc/nginx/sites-available/wordpress /etc/nginx/sites-enabled/
sudo rm /etc/nginx/sites-enabled/default
sudo nginx -t
sudo systemctl restart nginx
```

## PHP Configuration

Configure PHP-FPM:
```bash
sudo nano /etc/php/8.1/fpm/pool.d/www.conf
```

Ensure these settings are correct:
```ini
user = www-data
group = www-data
listen.owner = www-data
listen.group = www-data
listen.mode = 0660
```

Restart PHP-FPM:
```bash
sudo systemctl restart php8.1-fpm
```

## WordPress Setup

1. Access your WordPress site at `http://192.168.56.13`
2. Complete the installation wizard
3. For manual configuration:
   ```bash
   cd /var/www/wordpress
   sudo cp wp-config-sample.php wp-config.php
   sudo nano wp-config.php
   ```
   Update the database credentials to match what you created earlier.

## Daily Workflow

1. Start your environment:
   ```bash
   vagrant up
   ```

2. SSH into the VM (if needed):
   ```bash
   vagrant ssh
   ```

3. Develop in your local project folder (changes sync automatically)

4. When done working:
   ```bash
   vagrant suspend
   ```

## Troubleshooting

Common issues and solutions:

1. **502 Bad Gateway**:
   - Check PHP-FPM is running: `sudo systemctl status php8.1-fpm`
   - Verify socket permissions: `ls -la /var/run/php/php8.1-fpm.sock`

2. **File Not Found**:
   - Check file permissions: `sudo chown -R www-data:www-data /var/www/wordpress`
   - Verify Nginx root directory in config

3. **Database Connection Issues**:
   - Verify MySQL user privileges
   - Check `wp-config.php` credentials

4. **Nginx Configuration Errors**:
   - Test config: `sudo nginx -t`
   - Check logs: `sudo tail -50 /var/log/nginx/error.log`

For quick PHP testing:
```bash
echo "<?php phpinfo(); ?>" | sudo tee /var/www/wordpress/test.php
```
(Remember to remove this file after testing)


# NGINX BASIC
`/etc/nginx/`  
The /etc/nginx/ directory is the default configuration root

`/etc/nginx/nginx.conf` 
The /etc/nginx/nginx.conf file is the default configuration entry point 

`/etc/nginx/conf.d/`
The /etc/nginx/conf.d/ directory contains the default HTTP server configuration