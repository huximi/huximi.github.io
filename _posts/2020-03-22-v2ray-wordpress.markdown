---
layout:     post
title:      一键安装v2ray
subtitle:   一键安装v2ray + WordPress
date:       2020-03-22
author:     guyang
header-img: "img/post-bg-2018.jpg"
tags:    
    - v2ray
    - Mysql
    - PHP7
    - WordPress
    - 科学上网
    - 一键脚本    
---

### 域名设置：

#### 购买VPS
- **1. 购买云服务器**

国内的推荐[腾讯云](https://cloud.tencent.com/redirect.php?redirect=1005&cps_key=42c9b322fd48ff0ce405a0c7d78612fd)，毕竟大公司，工单服务贼及时！还送免费的CDN加速流量~
国外的Vultr还不错，全球15个机房中心，采用小时计费策略，需要国外服务器的也可以尝试下~[Vultr购买图解步骤](https://www.flyzy2005.cn/vps/vultr-deploy)
免费试用的话，可以选择[谷歌GCP](https://console.cloud.google.com),有mail账号就可以免费一年的服务器.

- **2.购买域名**

有了云服务器，还需要一个域名。国内的域名需要备案，购买的话阿里云腾讯云都可以；
国外的[namesilo](https://www.namesilo.com)也可以试试~但是可能国外的域名DNS解析会比较慢。


#### DNS代理

在云服务器管理页面设置 DNS服务器,使用 [cloudflare](https://dash.cloudflare.com) 进行DNS代理；如下：

```
Type	Value
NS	carter.ns.cloudflare.com
NS	elle.ns.cloudflare.com
```

#### **DNS解析设置**
如域名是 xxx.com ; ip是 xx.xx.xx.xx; 
设置A记录：
```
Type	Name	        Content	       TTL	   Proxy status	
A       xxx.com         xx.xx.xx.xx    Auto        DNS only
A       tls             xx.xx.xx.xx    Auto        DNS only
A       www             xx.xx.xx.xx    Auto        DNS only
```

### 环境安装：

#### 一键安装v2ray：
```
wget -N --no-check-certificate -q -O install.sh "https://raw.githubusercontent.com/wulabing/V2Ray_ws-tls_bash_onekey/master/install.sh" && chmod +x install.sh && bash install.sh
```
#### 安装MySQL:
```
apt-get install mysql-server
```
#### 安装PHP7:
```
apt-get install php-fpm php-mysql
```
#### 配置Nginx使用PHP7:
 - 修改Nginx的配置文件：
 
```
vi /etc/nginx/conf/conf.d/v2ray.conf
```

 - 添加nginx对PHP的处理，修改后的配置文件如下所示：
 
```
 server {
        listen 443 ssl http2;
        ssl_certificate       /data/v2ray.crt;
        ssl_certificate_key   /data/v2ray.key;
        ssl_protocols         TLSv1.2 TLSv1.3;
        ssl_ciphers           TLS13-AES-256-GCM-SHA384:TLS13-CHACHA20-POLY1305-SHA256:TLS13-AES-128-GCM-SHA256:TLS13-AES-128-CCM-8-SHA256:TLS13-AES-128-CCM-SHA256:EECDH+CHACHA20:EECDH+CHACHA20-draft:EECDH+ECDSA+AES128:EECDH+aRSA+AES128:RSA+AES128:EECDH+ECDSA+AES256:EECDH+aRSA+AES256:RSA+AES256:EECDH+ECDSA+3DES:EECDH+aRSA+3DES:RSA+3DES:!MD5;
        server_name www.xxx.com;
        index index.php index.html index.htm;
        root  /home/wwwroot/3DCEList;
        error_page 400 = /400.html;
        
        location /81ab9590/
        {
        proxy_redirect off;
        proxy_pass http://127.0.0.1:30993;
        proxy_http_version 1.1;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        proxy_set_header Host $http_host;
        }
        
        location /
        {
        	try_files $uri $uri/ /index.php?$args;
        }
        
        rewrite /wp-admin$ $scheme://$host$uri/ permanent;
        
        location ~ \.php$ {
        	fastcgi_pass unix:/run/php/php7.0-fpm.sock;
        	fastcgi_index index.php;
        	fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        	include fastcgi_params;
        }
 }
 server {
        listen 80 default;
        server_name _;
        server_name www.xxx.com;
        return 301 https://www.xxx.com$request_uri;
 }

```
#### 重启Nginx启动新配置文件:
```
systemctl restart nginx
```

### 下载WordPress:
```
wget http://wordpress.org/latest.tar.gz
```
#### 解压：
```
tar -xzvf latest.tar.gz
```
### 创建WordPress操作的数据库和用户:

#### root密码登录MySQL：
```
mysql -u root -p
```
#### 创建数据库：
```
CREATE DATABASE wordpress;
```
#### 创建用户：
```
CREATE USER wordpress@localhost;
```
#### 设置密码：
```
SET PASSWORD FOR wordpress@localhost=PASSWORD("wordpress");
```
#### 配置权限：
```
GRANT ALL PRIVILEGES ON wordpress.* TO wordpress@localhost IDENTIFIED BY 'wordpress';
```
#### 刷新权限配置：
```
FLUSH PRIVILEGES;
```
#### 退出MySQL：
```
QUIT;
```

### 配置WordPress:
重命名示例文件wp-config（此处的路径/root/wordpress对应你自己的存放路径）
```
mv /root/wordpress/wp-config-sample.php /root/wordpress/wp-config.php

vim /root/wordpress/wp-config.php


https://api.wordpress.org/secret-key/1.1/salt/
```

#### 配置Nginx:
```
cp -r /root/wordpress/* /home/wwwroot/3DCEList
```
#### 修改权限：
```
chown -R www-data:www-data /home/wwwroot/3DCEList
```
#### 重启Nginx:
```
systemctl restart nginx
```

### 配置WordPress
```
https://www.xxx.com/wp-admin/install.php
```
`