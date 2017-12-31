# Solder
Installing Technic Solder


Install php7 repository:

```
sudo apt-get install software-properties-common python-software-properties
sudo add-apt-repository ppa:ondrej/php
sudo apt-get update
sudo apt-get install php7.0
```

Install PHP 7.0

```
sudo apt-get install php7.0 php7.0-bcmath php7.0-xml php7.0-mbstring php7.0-mysql php7.0-sqlite php7.0-curl php7.0-json php7.0-zip php7.0-gd php7.0-mcrypt php7.0-fpm
```

Create the directory where solder will live

```
mkdir -p /var/www
cd /var/www
```

Download solder

```
git clone -b dev https://github.com/TechnicPack/TechnicSolder.git solder
```

Make the repository directory

```
mkdir -p /var/www/repo
```

Install and configure MySQL server. Be sure to remember the password you set.

```
sudo apt-get install mysql-server

mysql -p
CREATE DATABASE solder;
exit
```

Copy sample configuration file

```
cp -rf /var/www/solder/app/config-sample /var/www/solder/app/config
```

Edit solder.php

```
nano /var/www/solder/app/config/solder.php
```
Change "repo_location" to "/var/www/solder/public/"
Change "mirror_url" to "http://solder.example.com/" 

Configure app.php

```
nano /var/www/solder/app/config/app.php
```
Change "url" to "http://solder.example.com/"

Configure database.php

```
nano /var/www/solder/app/config/database.php
```
Change "default" to "mysql"
Change "username" to "root"
Change "password" to the password you set when installing MySQL

Finished configuring solder files

Change into solder directory

```
cd /var/www/solder
```

Download and install composer

```
wget https://getcomposer.org/download/1.4.2/composer.phar
php7.0 composer.phar install
```

Migrate databases

```
php7.0 artisan migrate:install
php7.0 artisan migrate
```

Configure webserver

```
nano /etc/nginx/sites-available/solder
```
Paste this in and change "solder.example.com"
```
server {
    listen 80;
    listen [::]:80;
    server_name solder.example.com;
    root /var/www/solder/public/;
    index index.php;

    access_log /var/log/nginx/solder-access.log;
    error_log  /var/log/nginx/solder-error.log error;

    # allow larger file uploads and longer script runtimes
    client_max_body_size 100m;
    client_body_timeout 120s;

    sendfile off;
    location / {
            try_files $uri $uri/ /index.php?$query_string;
    }

    location ~ \.php$ {
        fastcgi_split_path_info ^(.+\.php)(/.+)$;
        fastcgi_pass unix:/var/run/php/php7.0-fpm.sock;
        fastcgi_index index.php;
        include fastcgi_params;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        fastcgi_intercept_errors off;
        fastcgi_buffer_size 16k;
        fastcgi_buffers 4 16k;
        fastcgi_connect_timeout 300;
        fastcgi_send_timeout 300;
        fastcgi_read_timeout 300;
        include /etc/nginx/fastcgi_params;
    }
}
```

Copy the site and repo

```
ln -s /etc/nginx/sites-available/solder /etc/nginx/sites-enabled/solder
ln -s /var/www/repo /var/www/solder/public/mods
```

Assign correct permissions

```
chown -R www-data /var/www
```

You're finished! Restart nginx and php-fpm

```
service nginx restart
service php7.0-fpm restart
```

NOTE: If nginx fails to restart, you may have apache installed, to remove it:
```
sudo apt-get purge apache2
```
Then restart nginx
```
service nginx restart
```
