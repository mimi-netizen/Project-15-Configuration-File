#!/bin/bash

exec > /var/log/tooling.log 2>&1

# Create directory and mount EFS

mkdir -p /var/www/
sudo mount -t efs -o tls,accesspoint=fsap-09c86085067becad5 fs-09f8367a7f9f0857c:/ efs /var/www/

# Install and start Apache

sudo yum install -y httpd
sudo systemctl start httpd
sudo systemctl enable httpd

# Install PHP and necessary extensions

sudo yum module reset php -y
sudo yum module enable php:remi-7.4 -y
sudo yum install -y php php-common php-mbstring php-opcache php-intl php-xml php-gd php-curl php-mysqlnd php-fpm php-json
sudo systemctl start php-fpm
sudo systemctl enable php-fpm

# Clone the tooling repository

git clone https://github.com/mimi-netizen/tooling.git
mkdir -p /var/www/html/
cp -R tooling/html/\* /var/www/html/

# Setup MySQL database

cd /tooling
mysql -h cdk-rds.cly8ayoym3bc.us-west-1.rds.amazonaws.com -u admin -p 5wTMa=Gk+95?SQX < tooling-db.sql

cd /var/www/html/
touch healthstatus

sed -i "s/$db = mysqli_connect('mysql.tooling.svc.cluster.local', 'admin', 'admin', 'tooling');/$db = mysqli_connect('cdk-rds.cly8ayoym3bc.us-west-1.rds.amazonaws.com', 'admin', '5wTMa=Gk+95?SQX', 'toolingdb');/g" functions.php

chcon -t httpd_sys_rw_content_t /var/www/html/ -R

# Disable Apache welcome page and restart Apache

sudo mv /etc/httpd/conf.d/welcome.conf /etc/httpd/conf.d/welcome.conf_backup
sudo systemctl restart httpd
