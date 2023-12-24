---
title: "CentOS下使用Docker搭建LNMP开发环境"
date: 2023-12-10T20:51:53+08:00
categories: ['Docker']
tags: ['Docker','CentOS','PHP']
draft: true
---
## 运行环境
- 操作系统：CentOS Linux 8
- Docker：Docker Engine - Community 24.0.7  
## 参考网站  
- [Docker如何打包软件](https://docs.docker.com/build/building/packaging/)
- [PHP镜像](https://hub.docker.com/_/php)  
- [MySQL镜像](https://hub.docker.com/_/mysql)  
- [Nginx镜像](https://hub.docker.com/_/nginx)  
- [Dockerfile reference](https://docs.docker.com/engine/reference/builder/#from)  
## 具体操作  
### 安装PHP镜像  
```bash
docker pull php:7.4.33-fpm
```
### 安装MySQL镜像  
```bash
docker pull mysql:8.2.0
```
### 安装Nginx镜像  
```bash
docker pull nginx:stable-alpine3.17-slim
```
### 创建Dockerfile  
```bash
mkdir -p /home/dockerfile
cd /home/dockerfile
touch Dockerfile
```
### 编辑Dockerfile  
```bash
vim Dockerfile
```
```dockerfile
# 使用 PHP 7.4.33 FPM 版本作为基础镜像
FROM php:7.4.33-fpm

# 安装必要的 PHP 扩展
RUN docker-php-ext-install mysqli pdo pdo_mysql

# 将你的 PHP 代码复制到容器中
COPY . /var/www/html

# 设置工作目录
WORKDIR /var/www/html

# 暴露端口，这应该与你的 PHP-FPM 配置中的监听端口一致
EXPOSE 9000

# 启动 PHP-FPM 服务
CMD ["php-fpm"]
```
然后，你需要创建一个 Nginx 配置文件，例如 default.conf，来配置 Nginx 服务器：
```nginx
server {
    listen 80;
    server_name localhost;

    location / {
        root /var/www/html;
        index index.php index.html;
        try_files $uri $uri/ /index.php?$args;
    }

    location ~ \.php$ {
        root /var/www/html;
        fastcgi_pass php:9000;
        fastcgi_index index.php;
        include fastcgi_params;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
    }
}
```
然后，你可以创建一个 docker-compose.yml 文件来定义你的服务：
```yaml
version: '3'
services:
  web:
    image: nginx:stable-alpine3.17-slim
    volumes:
      - ./site:/var/www/html
      - ./default.conf:/etc/nginx/conf.d/default.conf
    ports:
      - "8080:80"
    depends_on:
      - php
  php:
    build: .
    volumes:
      - ./site:/var/www/html
  mysql:
    image: mysql:8.2.0
    environment:
      MYSQL_ROOT_PASSWORD: secret
      MYSQL_DATABASE: dbname
      MYSQL_USER: user
      MYSQL_PASSWORD: secret
    volumes:
      - ./mysql:/var/lib/mysql
```