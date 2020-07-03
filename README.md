Tugas Day 12 

1. Clone repository absensi siswa
# sudo yum install git
# git clone https://gitlab.com/muhammadyaqin/laravel5_sample.git laravel5/

2. Install docker dan docker-compose
# sudo yum remove docker \
   docker-client \
   docker-client-latest \
   docker-common \
   docker-latest \
   docker-latest-logrotate \
   docker-logrotate \
   docker-engine
#  sudo yum install -y yum-utils
# sudo yum-config-manager \
   --add-repo \
   https://download.docker.com/linux/centos/docker-ce.repo
#  sudo yum update -y
 # sudo yum install docker-ce docker-ce-cli containerd.io
 # sudo usermod -aG docker centos
 # sudo systemctl start docker
3. Add dockerfile dan docker-compose file
# nano app.dockerfile
FROM php:7.2-fpm

RUN apt-get update && apt-get install -y libmcrypt-dev && apt clean
RUN pecl install mcrypt-1.0.2 && docker-php-ext-enable mcrypt
RUN apt-get install -y libfreetype6-dev libjpeg62-turbo-dev libmcrypt-dev libpng-dev && apt clean
RUN docker-php-ext-install -j$(nproc) iconv pdo_mysql 
RUN docker-php-ext-configure gd --with-freetype-dir=/usr/include/ --with-jpeg-dir=/usr/include/
RUN docker-php-ext-install -j$(nproc) gd 

WORKDIR /var/www
# nano web.dockerfile
FROM nginx:latest

ADD ./laravelVHost.conf /etc/nginx/conf.d/default.conf
WORKDIR /var/www
# nano docker-compose.yml
version: '3'

services:
  web:
    build:
      context: ./
      dockerfile: web.dockerfile
    volumes:
      - ./:/var/www
    restart: always
    ports:
      - "8080:80"
    links:
      - app

  app:
    build:
      context: ./
      dockerfile: app.dockerfile
    volumes:
      - ./:/var/www
    restart: always
    links:
      - database
    environment:
      - "DB_PORT=3306"
      - "DB_HOST=database"
  
  database:
    image: mysql:latest
    hostname: database
    restart: always
    command: --default-authentication-plugin=mysql_native_password
    environment:
        MYSQL_ROOT_PASSWORD: secret
        MYSQL_DATABASE: laravelDB
    ports:
        - "33061:3306"

  cache:
    image: redis:latest
    ports: 
      - "63791:6379"
# nano laravelVHost.conf
server {
    listen 80;
    index index.php index.html;
    root /var/www/public;

    location / {
        try_files $uri /index.php?$args;
    }

    location ~ \.php$ {
        fastcgi_split_path_info ^(.+\.php)(/.+)$;
        fastcgi_pass app:9000;
        fastcgi_index index.php;
        include fastcgi_params;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        fastcgi_param PATH_INFO $fastcgi_path_info;
    }
}

4. Jalankan docker compose
# docker-compose up -d
5. Install composer
# sudo yum install php-cli php-zip wget unzip
# php -r "copy('https://getcomposer.org/installer', 'composer-setup.php');"
# HASH="$(wget -q -O - https://composer.github.io/installer.sig)"
# php -r "if (hash_file('SHA384', 'composer-setup.php') === '$HASH') { echo 'Installer verified'; } else { echo 'Installer corrupt'; unlink('composer-setup.php'); } echo PHP_EOL;"
# composer install

6. Setting environment
# cp .env.example .env
# nano .env
…
DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=33061:33061
DB_DATABASE=laravelDB
DB_USERNAME=root
DB_PASSWORD=secret
...
# php artisan key:generate
# php artisan migrate
 #  chmod -R 777 storage/
 # chmod -R 777 bootstrap/

7. Buka dari browser dengan port 8080


