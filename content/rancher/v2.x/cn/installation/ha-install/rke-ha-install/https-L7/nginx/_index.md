---
title: Nginx配置
weight: 1
---

## 一、安装Nginx

NGINX拥有所有主流操作系统的软件包，通过包管理器可以很轻松安装。有关NGINX安装帮助，请参考[nginx安装文档](https://www.nginx.com/resources/wiki/start/topics/tutorials/install/).

### 1、创建Nginx配置

在安装nginx之前，需要先创建rancher代理配置文件`/etc/nginx/conf.d/rancher.conf`。

- 复制粘贴以下文件到编辑器，并保存到 `/etc/nginx/conf.d/rancher.conf`.

    **NGIN示例配置:**

    ```bash
    upstream rancher {
        server IP_NODE_1:80;
        server IP_NODE_2:80;
        server IP_NODE_3:80;
    }

    map $http_upgrade $connection_upgrade {
        default Upgrade;
        ''      close;
    }

    server {
        listen 443 ssl http2;
        server_name rancher.yourdomain.com;
        ssl_certificate /etc/your_certificate_directory/fullchain.pem;
        ssl_certificate_key /etc/your_certificate_directory/privkey.pem;

        location / {
            proxy_set_header Host $host;
            proxy_set_header X-Forwarded-Proto $scheme;
            proxy_set_header X-Forwarded-Port $server_port;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_pass http://rancher;
            proxy_http_version 1.1;
            proxy_set_header Upgrade $http_upgrade;
            proxy_set_header Connection $connection_upgrade;
            # This allows the ability for the execute shell window to remain open for up to 15 minutes. Without this     parameter, the default is 1 minute and will automatically close.
            proxy_read_timeout 900s;
            proxy_buffering off;
        }
    }

    server {
        listen 80;
        server_name rancher.yourdomain.com;
        return 301 https://$server_name$request_uri;
    }
    ```

    >为了减少网络传输的数据量，可以在七层代理的`http`定义中添加`GZIP`功能。

    ```bash
    # Gzip Settings
    gzip on;
    gzip_disable "msie6";
    gzip_disable "MSIE [1-6]\.(?!.*SV1)";
    gzip_vary on;
    gzip_static on;
    gzip_proxied any;
    gzip_min_length 0;
    gzip_comp_level 8;
    gzip_buffers 16 8k;
    gzip_http_version 1.1;
    gzip_types
      text/xml application/xml application/atom+xml application/rss+xml application/xhtml+xml image/svg+xml     application/font-woff
      text/javascript application/javascript application/x-javascript
      text/x-json application/json application/x-web-app-manifest+json
      text/css text/plain text/x-component
      font/opentype application/x-font-ttf application/vnd.ms-fontobject font/woff2
      image/x-icon image/png image/jpeg;
    ```

- 在`/etc/nginx/conf.d/rancher.conf`中, 替换 `IP_NODE_1`, `IP_NODE_2`,  `IP_NODE_3` 为需要添加到集群的Linux主机的IP；

- 在`/etc/nginx/conf.d/rancher.conf`中, 替换`FQDN`为你设置用来登录rancher的域名；

- 在`/etc/nginx/conf.d/rancher.conf`中, 替换`/certs/fullchain.pem`为证书的路径；

- 在`/etc/nginx/conf.d/rancher.conf`中, 替换`/certs/privkey.pem`为证书密钥的路径；

## 二、运行NGINX

- 重新加载或者重启NGINX

    ```bash
    # Reload NGINX
    nginx -s reload
    # Restart NGINX
    # Depending on your Linux distribution
    service nginx restart
    systemctl restart nginx
    ```

## 三、访问Rancher UI

安装成功后，通过`https://FQDN`来访问RANCHER UI
