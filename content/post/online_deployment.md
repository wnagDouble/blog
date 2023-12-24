---
title: "前后端分离PHP项目线上部署记录"
date: 2023-12-23T18:05:16+08:00
categories: ['Nginx']
draft: false
---
>前提是已具备基本LNMP环境。以后端的视角记录。主要是Nginx配置。
## 背景
项目所在目录：`/var/www/html/project_name/`  
前端项目所在目录：`/var/www/html/project_name/web/`  
后端项目所在目录：`/var/www/html/project_name/api/`
## 前端部署
在`/var/www/html/project_name/web/`下解压缩前端提供的压缩包。
### 配置nginx
```nginx
server {
    listen 80;
    server_name domain.com www.domain.com;
    return 301 https://www.domain.com$request_uri;
}

server {
    listen 443 ssl http2; #【1】
    server_name  www.domain.com domain.com; # 域名设置
    access_log   /var/log/nginx/www.domain.com-access.log;
    error_log   /var/log/nginx/www.domain.com-error.log;
    index index.html index.htm; # 日志目录

    ssl_certificate /etc/letsencrypt/live/domain.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/domain.com/privkey.pem;

    client_max_body_size 5m;

    root /var/www/html/project_name/web;
    index index.html index.htm;

    location / {
	try_files $uri $uri/ /index.html;#【2】
    }


    location ^~ /api/ {#【3】
        add_header access-control-allow-origin *;

        proxy_pass  http://127.0.0.1:8082;#将api路径下的请求转发到后端
        proxy_connect_timeout 1800;
        proxy_read_timeout 1800;
        proxy_send_timeout 1800;

        proxy_set_header X-Forwarded-Host $host:$server_port;
        proxy_set_header  X-Real-IP  $remote_addr;
        proxy_set_header    Origin        $host:$server_port;
        proxy_set_header    Referer       $host:$server_port;

        proxy_set_header Accept-Encoding "";

        # 去除/api/前缀
        rewrite ^/api/(.*)$ /$1 break;
    }
}
```  
- 注释【1】：让Nginx在443端口上监听HTTPS请求，并使用SSL/TLS协议加密通信内容，同时使用HTTP/2协议。
- 注释【2】：首先，Nginx会尝试将请求的URI作为文件路径，如果这个文件存在，就直接返回这个文件的内容。  
  如果文件不存在，Nginx会尝试将请求的URI作为目录路径，如果这个目录存在，就返回这个目录的默认页面。  
  如果文件和目录都不存在，Nginx会返回/index.html这个文件的内容。
- 注释【3】：`^~`是一个匹配修饰符，表示如果这个location的前缀匹配到请求，那么就立即停止搜索其他location，处理这个请求。  
  `/api/`是location的匹配规则，表示所有以/api/开头的请求都会被这个location处理。  
  在这个location块中，所有以/api/开头的请求都会被反向代理到http://127.0.0.1:8888，并且/api/前缀会被去除。这是通过proxy_pass和rewrite两个指令实现的。  
## 后端部署  
在`/var/www/html/project_name/api/`下上传代码。 
### 配置nginx  
这个后端的Nginx配置写得不好。server里实际可以不需要指定二级域名（api.domain.com)，只需要指定端口就可以了。这个二级域名在项目实际运行当中，并没有直接使用。
后端的接口调用都是通过前端的Nginx转发的。这个域名没用但解析了，不大安全。改进方法待定。
```nginx
server {
    listen 80;
    server_name api.domain.com;
    return 301 https://$host$request_uri;
}

server {
    listen 443 ssl http2;
    server_name api.domain.com;

    # Certificates
    ssl_certificate /etc/letsencrypt/live/domain.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/domain.com/privkey.pem;

    access_log   /var/log/nginx/api.domain.com-access.log;
    error_log   /var/log/nginx/api.domain.com-error.log;

    location / {
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Host $host;

        proxy_pass http://localhost:8082;
    }
}

server {
    listen       8082;
    server_name  api.domain.com;
    root         /var/www/html/project_name/api/public;
    charset      utf-8;
    autoindex    off;

    location /
    {
        add_header Access-Control-Allow-Origin  '*';
        add_header Access-Control-Allow-Methods 'GET, POST, OPTIONS, PUT, DELETE';
        add_header Access-Control-Allow-Headers 'x-token,DNT,X-Mx-ReqToken,Keep-Alive,User-Agent,X-Requested-With,If-Modified-Since,Cache-Control,Authorization';

        if ($request_method = 'OPTIONS')
        {
            return 204;
        }

        proxy_set_header    Host                $http_host;
        proxy_set_header    X-Real-IP           $remote_addr;
        proxy_set_header    X-Real-PORT         $remote_port;
        proxy_set_header    X-Forwarded-For     $proxy_add_x_forwarded_for;

        location / {
            # !-e 判断的是目录或文件是否不存在，不存在时则重写
            # !-f 判断的是文件是否不存在，不存在时则重写
            if (!-f $request_filename) {
                # 路由重写 - 一律重写至 index.php 下，由项目代码进行路由定义
                rewrite ^/(.*)$ /index.php?s=$1 last; break;
            }
        }

        location ~ \.php$ {
            include                  fastcgi_params;
            include                  fastcgi.conf;
            fastcgi_pass             fastcgi_backend;
            fastcgi_intercept_errors on;
            fastcgi_param SCRIPT_FILENAME $document_root/$fastcgi_script_name;
        }
    }
}
```

## 遇到的问题
### 如何将文件上传到服务器
我文件上传一般习惯用xftp,但是project_name目录下的文件夹权限是www-data，而我当前的登录用户名是ubuntu。所以我用xftp上传文件时会报错。  
**解决方法**：切换到www-data用户，给自己添加认证信息，然后再用xftp上传文件。
```bash
sudo su www-data
vim ~/.ssh/authorized_keys
```
将自己的公钥添加到authorized_keys文件中。  
**PS:** 如果登录方式不是SSH可以略过。  
### 需要一个SSL证书  
**解决方法**：使用[Let's Encrypt](https://letsencrypt.org/)免费SSL证书。


