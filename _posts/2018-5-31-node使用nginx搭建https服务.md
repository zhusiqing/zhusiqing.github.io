---
layout:     post
title:      node使用nginx反向代理搭建https服务
subtitle:   node,nginx,反向代理,http,https
date:       2018-05-31
author:     siqing
header-img: img/post-bg-debug.png
catalog: true
tags:
    - node
    - nginx
    - http
    - https
---

# `node` 使用 `nginx` 反向代理搭建https服务

> 使用自己生成的证书

## `nginx` http配置

1. 编辑`nginx.conf`

```bash
vim /usr/local/etc/nginx/nginx.conf
```

修改 http 里面的server

```bash
server {
   #将原来的8080改成80端口，这样就能隐藏请求中的端口号了
   listen       80;
   #这里改成你想要的测试域名
   server_name  www.test.com;
   
   server_name_in_redirect off;
   proxy_set_header Host $host:$server_port;
   proxy_set_header X-Real-IP $remote_addr;
   proxy_set_header REMOTE-HOST $remote_addr;
   proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
   
   location / {
        #需要代理的地址
        proxy_pass http://127.0.0.1:3000/;
    }
}
```

2. 重启`nginx`

```bash
nginx -s reload
```

3. 修改hosts文件

```bash
sudo vim /etc/hosts
```

添加

```bash
127.0.0.1 www.test.com
```

## `nginx` https配置 

1. 利用`openssl`生成证书

```bash
cd /usr/local/etc/nginx
# 设置server.key
openssl genrsa -des3 -out server.key 1024
# 参数设置
openssl req -new -key server.key -out server.csr
# 写RSA密钥
openssl rsa -in server.key -out server_nopwd.key
# 获取密钥
openssl x509 -req -days 365 -in server.csr -signkey server_nopwd.key -out server.crt
```

2. 修改`nginx`配置文件`nginx.conf`

```bash
server {
   #将原来的8080改成80端口，这样就能隐藏请求中的端口号了
   listen       80;
   #这里改成你想要的测试域名
   server_name  www.test.com;
   
   # ------------ 增加的 ------------
   ssl on;
   #你的证书地址
   ssl_certificate /usr/local/etc/nginx/server.crt;
   #私钥地址
   ssl_certificate_key /usr/local/etc/nginx/server_nopwd.key;
   # ------------       -------------
   
   server_name_in_redirect off;
   proxy_set_header Host $host:$server_port;
   proxy_set_header X-Real-IP $remote_addr;
   proxy_set_header REMOTE-HOST $remote_addr;
   proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
   
   location / {
        #需要代理的地址
        proxy_pass http://127.0.0.1:3000/;
    }
}
```

修改 https 下面的server

> 把https server相关注释打开

```bash
# HTTPS server
#
server {
    listen       443 ssl;
    server_name  localhost;

#   ssl_certificate      cert.pem;
#   ssl_certificate_key  cert.key;
    ssl_certificate /usr/local/etc/nginx/server.crt;
    ssl_certificate_key /usr/local/etc/nginx/server_nopwd.key;

#   ssl_session_cache    shared:SSL:1m;
#   ssl_session_timeout  5m;

#   ssl_ciphers  HIGH:!aNULL:!MD5;
#   ssl_prefer_server_ciphers  on;
   server_name_in_redirect off;
   proxy_set_header Host $host:$server_port;
   proxy_set_header X-Real-IP $remote_addr;
   proxy_set_header REMOTE-HOST $remote_addr;
   proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;

    location / {
        proxy_pass http://127.0.0.1:3000/;
                root   html;
          #      index  index.html index.htm;
    }
}
```

# `nginx` gzip 开启

编辑 `nginx.conf`


```bash
http {

    sendfile            on;
    tcp_nopush          on;
    tcp_nodelay         on;
    gzip                on;
    # 压缩等级
    gzip_comp_level     5;
    gzip_min_length 1000;
    gzip_proxied    expired no-cache no-store private auth;
    # 要压缩类型
    gzip_types      application/javascript text/css;
    keepalive_timeout   65;
    types_hash_max_size 2048;
}
```

# `node` 使用 `nginx` 反向代理搭建https服务

> 使用CA证书

## node服务器

> 基于express

```js
const http = reuqire('http')
const https = require('https')
const path = require('path')
const fs = require('fs')
const express = require('express')

const app = express()

const option = {
    key: fs.readFileSync('**/**/**.key'),
    cert: fs.readFileSync('**/**/**.pem')
}
// http和https都支持的
http.createServer(app).listen(80)
https.createServer(option, app).listen(443)
```

## nginx 反向代理`http` node服务成 `https`

> nginx.conf

```
server {
    listen 443;
    server_name www.zsqlm.cn;
    ssl on;
    index index.html index.htm;
    ssl_certificate   **.pem;
    ssl_certificate_key  **.key;
    ssl_session_timeout 5m;
    ssl_ciphers ECDHE-RSA-AES128-GCM-SHA256:ECDHE:ECDH:AES:HIGH:!NULL:!aNULL:!MD5:!ADH:!RC4;
    ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
    ssl_prefer_server_ciphers on;
    location / {
        #node.js应用的端口
        proxy_pass http://127.0.0.1:9901;
        proxy_set_header Host $host:$server_port;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header REMOTE-HOST $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
  }
}
```
