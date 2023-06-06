---
layout:     post
title:      Centos7中以yum方式安装nginx
subtitle:   Centos7中以yum方式安装nginx
date:       2023-06-06
author:     guyang
header-img: "img/post-bg-2018.jpg"
tags:    
    - centos7
    - nginx
---

# Centos7中以yum方式安装nginx

## 1、添加`CentOS 7` `Nginx` `yum`资源库
```
rpm -Uvh http://nginx.org/packages/centos/7/noarch/RPMS/nginx-release-centos-7-0.el7.ngx.noarch.rpm
```

## 2、安装`nginx`
```
yum -y install nginx   //安装nginx
```

## 3、启动`nginx`
修改配置文件，以`root`用户启动
```
vi /etc/nginx/nginx.conf  

#修改为root 启动
user root;
worker_processes auto;
error_log /var/log/nginx/error.log;
pid /run/nginx.pid;

# Load dynamic modules. See /usr/share/doc/nginx/README.dynamic.
include /usr/share/nginx/modules/*.conf;

events {
    worker_connections 1024;
}

http {
    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

    access_log  /var/log/nginx/access.log  main;

    sendfile            on;
    tcp_nopush          on;
    tcp_nodelay         on;
    keepalive_timeout   65;
    types_hash_max_size 4096;

    include             /etc/nginx/mime.types;
    default_type        application/octet-stream;

    # Load modular configuration files from the /etc/nginx/conf.d directory.
    # See http://nginx.org/en/docs/ngx_core_module.html#include
    # for more information.
    include /etc/nginx/conf.d/*.conf;

    server {
        listen       80;
        listen       [::]:80;
        server_name  _;
        root         /usr/share/nginx/html;

        # Load configuration files for the default server block.
        include /etc/nginx/default.d/*.conf;

        error_page 404 /404.html;
        location = /404.html {
        }

        error_page 500 502 503 504 /50x.html;
        location = /50x.html {
        }
    }
}
```

启动nginx
```
systemctl start nginx
```
开机启动
```
systemctl enable nginx
```

## 4、yum方式安装的默认地址和配置的默认地址
```
/etc/nginx/nginx.conf  //yum方式安装后默认配置文件的路径

/usr/share/nginx/html  //nginx网站默认存放目录

/usr/share/nginx/html/index.html //网站默认主页路径
```

## 5、配置其他端口的代码

### 5.1、端口转发：
```
cd /etc/nginx/conf.d
cp default.conf 8080_localhost.conf
vi 8080_localhost.conf
```
8080_localhost.conf 的内容如下：
```
server {
    listen            8080;
    server_name       localhost;
    access_log        /var/log/nginx/80.access.log;
    
	include /etc/nginx/conf.d/localhost/*.conf;
    location /docs/ {
      root /home/skynj/fonts;
      index  index.html;
    }
    
    location / {
      root /home/;
      index  index.html;
    }
    location ~ /demo/.*\.(html|htm|map|gif|jpg|jpeg|bmp|png|ico|txt|js                                                       |css|json|woff|ttf|TTF|bcmap|mp4|swf|svg)$ {
        root /home/huximi/front/;
        index  index.html;
        expires 2h;
    }
    location /demo/ {
        client_max_body_size        30m;
        client_body_buffer_size 2048k;
        proxy_max_temp_file_size 4096m;
        proxy_set_header Host $host:8095;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Real-Port $remote_port;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_connect_timeout 600s;
        proxy_send_timeout 1200;
        proxy_read_timeout 1200;
        # buffer
        proxy_buffering on;
        proxy_buffer_size 128k;
        proxy_buffers 8 2048k;
        proxy_busy_buffers_size 4096k;
        # 反代后台服务
        proxy_pass localhost:8081/demo/;  
    }
}
```

### 5.2、将域名转发到本地端口
例：访问 `sonar.huximi.cn` 时转发的地址为 `192.168.2.11:8020` ：

首先绑定域名和ip:
```
huximi.cn  192.168.2.11
```
在`/etc/nginx/conf.d/` 下增加`8020_localhost.conf`:
```
cd /etc/nginx/conf.d/
vi 8020_localhost.conf
```
如下：
```
server {
    listen       80;
    server_name  sonar.huximi.cn;
    index  index.php index.html index.htm;

    location / {
       proxy_pass  http://127.0.0.1:8020; # 转发规则
       proxy_set_header Host $proxy_host; # 修改转发请求头，让8020端口的应用可以受到真实的请求
       proxy_set_header X-Real-IP $remote_addr;
       proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }

    error_page   500 502 503 504  /50x.html;
    location = /50x.html {
        root   /usr/share/nginx/html;
    }
}

```

## 6、检查配置是否正确
```
nginx -t
```

## 7、重启`nginx`
```
systemctl restart nginx
```

## 8、如遇到`(13: Permission denied)`问题

则可能是 SELinux 策略限制：
如果您的系统启用了 SELinux，则可能会阻止 Nginx 访问其他端口或目录。您可以通过修改 SELinux 策略或使用 setsebool 命令来解决该问题。

### 8.1 SELinux 配置：

SELinux 是一种安全子系统，它在 Linux 操作系统中实现了访问控制策略。如果启用了 SELinux，可能会阻止 Nginx 访问其他端口或目录。在这种情况下，您可以通过以下步骤来解决问题：

1. 检查 SELinux 状态：运行以下命令以检查 SELinux 是否正在运行：

```
   sestatus
```

   如果输出结果为 "enabled"，则表示 SELinux 正在运行；如果输出结果为 "disabled"，则表示 SELinux 没有启用。

2. SELinux 策略配置：如果 SELinux 正在运行，则需要更新策略，以允许 Nginx 访问目标服务器上的应用程序。您可以使用 semanage、setsebool 或 chcon 命令来更新 SELinux 策略。

    - 使用 semanage 命令：

      ```
      semanage port -a -t http_port_t -p tcp 8020
      ```

      这将添加一个新的 TCP 端口规则，允许 Nginx 访问 8020 端口。

    - 使用 setsebool 命令：

      ```
      setsebool -P httpd_can_network_connect on
      ```

      这将启用 httpd_can_network_connect 参数，允许 Nginx 访问网络连接。

    - 使用 chcon 命令：

      ```
      chcon -R -t httpd_sys_content_t /path/to/your/app
      ```

      这将更新文件或目录的安全上下文，以允许 Nginx 访问应用程序目录。

3. 重新启动 Nginx：在更改 SELinux 策略后，需要重新启动 Nginx 才能使其生效。

通过以上步骤来解决 SELinux 策略限制可能导致的问题，并确保 Nginx 正常运行。
