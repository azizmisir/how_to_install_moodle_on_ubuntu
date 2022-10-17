# How To Install Moodle with Postgres, Nginx and PHP on an Ubuntu
## Step 1: Getting the packages
In order to run moodle some packages have to be installed first. Make sure to have up to date sources:
```command
sudo apt update
```
### Install Nginx:
```command
sudo apt install nginx graphviz aspell ghostscript clamav mlocate
```
### Install PostgreSQL:
```command
sudo apt install postgresql postgresql-contrib
```
### Install PHP and its dependencies:
```command
sudo add-apt-repository ppa:ondrej/php 
```
```command
sudo apt update 
```
```command
sudo apt install php7.4 php7.4-fpm php7.4-common php7.4-pgsql php7.4-gmp php7.4-curl php7.4-intl php7.4-mbstring php7.4-soap php7.4-xmlrpc php7.4-gd php7.4-xml php7.4-cli php7.4-zip unzip git curl
```
## Step 2: Installing Moodle
Nginx has decided that `wwwroot` is under `/var/www/html` which is fine for us. We will now install moodle there:
```command
mkdir downloads &&
cd downloads 
```
```command
wget https://download.moodle.org/stable400/moodle-4.0.4.tgz
```
```command
tar -zxvf moodle-4.0.4.tgz
```
```command
sudo cp moodle /var/www/html/ -R
cd 
```
```command
sudo chown www-data.www-data /var/www/html/moodle -R &&
sudo chmod -R 0755 /var/www/html/moodle
```
```command
sudo mkdir /var/www/moodledata &&
sudo chown www-data /var/www/moodledata -R &&
sudo chmod 0777 /var/www/moodledata 
```
## Step 3: Setting up the database
Now it's time to set up the database for moodle. To do so use the postgres user to create a new role called moodle which then will be able to handle the moodle database you are about to create:
```command
sudo -i -u postgres
psql
```
```command
CREATE USER moodle WITH PASSWORD 'yourpassword';
CREATE DATABASE moodle WITH OWNER moodle;
```
### Adjustments
```command
sudo nano /etc/php/7.4/fpm/php.ini
```

```php.ini
upload_max_filesize = 32M
max_execution_time = 300
memory_limit = 256M
post_max_size = 32M
max_input_time = 300
max_input_vars = 6000
date.timezone = CET
```

```command
sudo nano /etc/php/7.4/fpm/pool.d/www.conf
```

```www.conf
security.limit_extensions = .php
```

```command
sudo nano /etc/nginx/sites-available/default
```

```dafault
server {
        listen 80 default_server;
        listen [::]:80 default_server;
        root /var/www/html;
        index index.php index.html index.htm index.nginx-debian.html;
        server_name _;
        location / {
                try_files $uri $uri/ =404;
        }
        location ~ [^/]\.php(/|$) {
                fastcgi_split_path_info  ^(.+\.php)(/.+)$;
                fastcgi_index            index.php;
                fastcgi_pass unix:/var/run/php/php-fpm.sock;
                include fastcgi_params;
                fastcgi_param   PATH_INFO  $fastcgi_path_info;
                fastcgi_param   SCRIPT_FILENAME $document_root$fastcgi_script_name;
}
}
```

```command
sudo nginx -t &&
sudo systemctl restart php7.4-fpm  &&
sudo systemctl restart nginx  
```
## Step 4: Moodle Web Installer

```web
 http://yourdomain/moodle
 ```
## Step 5: Https & Let’s Encrypt SSL
```command
sudo apt install python3-certbot-nginx
```

```command
sudo certbot --nginx -d yourdomain
```

```command
sudo nano /var/www/html/moodle/config.php
```

```config.php
$CFG->wwwroot   = 'https://www.example.com/';
```
