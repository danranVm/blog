---
title: Nginx Quickstart
date: 2020-11-29
categories:
  - Nginx
tags:
  - Server
  - Nginx
---

::: tip
用 nginx 部署一个简单 web 项目: gizp, ssl 证书, http => https 等配置
:::

<!-- more -->

## 安装

本文以 `Ubuntu 20.04` 为例

```bash
sudo apt install nginx
```

## 常用命令

- 启动： `sudo service nginx start` 或者 `sudo systemctl start nginx`
- 重启：`restart`
- 重新加载配置：`reload`
- 强行重新加载配置：`force-reload`
- 查看状态：`status`
- 停止：`stop`

## 目录结构

- 配置文件目录: `/etc/nginx/`
- 配置文件入口：`/etc/nginx/nginx.conf`
- 通用配置目录：`/etc/nginx/conf.d/`
- 站点配置目录：`/etc/nginx/sites-available/`
  - 建议命名方式：`domain.com.conf`, 例如：`/etc/nginx/sites-available/danran.site.conf`
- 静态资源目录：`/var/www/`
  - 建议命名方式：`domain.com/`, 例如：`/var/www/danran.site/`
- 日志文件目录：`/var/log/nginx/`
  - 建议为每个服务器配置一个不同的 `access`和 `error`

## 配置文件

以 `danran.site` 为例，配置文件为：`/etc/nginx/sites-available/danran.site.conf`

```bash
# 权限
sudo chmod -R 777 /etc/nginx

cd /etc/nginx/sites-available
touch danran.site.conf
vim danran.site.conf
```

```vim
server {
	listen 80;
	listen [::]:80;

	server_name danran.site;

	root /var/www/danran.site;
	index index.html;

	location / {
		try_files $uri $uri/ =404;
	}
}
```

### gzip 配置

nginx 默认会开启 gzip, 默认配置如下：

```vim
gzip on; # 是否开启 gizp

# gzip_vary on;           # 是否在 header 中添加 Vary: Accept-Encoding
# gzip_proxied any;       # 反向代理是启用：any – 无条件压缩所有结果数据
# gzip_comp_level 6;      # 压缩级别
# gzip_buffers 16 8k;     # 缓冲区大小
# gzip_http_version 1.1;  # HTTP协议版本

# 压缩类型
# gzip_types text/plain text/css application/json application/javascript text/xml application/xml application/xml+rss text/javascript;
```

根据实际情况调整。  
另外建议调整一下 `gzip_min_length`

```vim
gzip_min_length 1024; # 启用 gzip 压缩的最小尺寸
```

完整版本 `/etc/nginx/conf.d/gzip.conf` 如下：

```vim
gzip               on;
gzip_vary          on;
gzip_proxied       any;
gzip_comp_level    6;
gzip_buffers       16 8k;
gzip_http_version  1.1;
gzip_min_length    1024;

gzip_types text/plain text/css application/json application/javascript text/xml application/xml application/xml+rss text/javascript;
```

## ssl 证书及 http 转发 https 配置

免费 ssl 证书来源：[acme.sh](https://github.com/acmesh-official/acme.sh/wiki/%E8%AF%B4%E6%98%8E)

```vim
server {
    listen       80;
    listen       [::]:80;
    server_name  danran.site;
    return       301 https://$host$request_uri;
}

server {
    listen       443 ssl http2;
    listen       [::]:443 ssl http2;
    server_name  danran.site;
    root         /usr/share/nginx/danran.site;
    index        index.html;

    ssl_certificate "/etc/pki/nginx/danran.site/cert.pem";
    ssl_certificate_key "/etc/pki/nginx/danran.site/key.pem";
    ssl_session_cache shared:SSL:1m;
    ssl_session_timeout  10m;
    ssl_ciphers PROFILE=SYSTEM;
    ssl_prefer_server_ciphers on;

    include /etc/nginx/conf.d/*.conf;

    location / {
        try_files $uri $uri/ =404;
    }

    error_page 404 /404.html;
        location = /40x.html {
    }
}
```
