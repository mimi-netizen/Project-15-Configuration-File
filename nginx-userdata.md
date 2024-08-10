#!/bin/bash

exec > /var/log/nginx.log 2>&1

yum install -y nginx
systemctl start nginx
systemctl enable nginx

git clone https://github.com/mimi-netizen/Project-15-Configuration-File.git

mv /Project-15-Configuration-File/reverse.conf /etc/nginx/
mv /etc/nginx/nginx.conf /etc/nginx/nginx.conf-distro

cd /etc/nginx/
touch nginx.conf

sed -n 'w nginx.conf' reverse.conf

systemctl restart nginx
rm -rf reverse.conf
rm -rf /Project-15-Configuration-File
