---
title: Nginx配置
weight: 1
---

## 安装Nginx

    NGINX拥有所有主流操作系统的软件包，通过包管理器可以很轻松安装。有关NGINX安装帮助，请参考[nginx安装文档](https://www.nginx.com/resources/wiki/start/topics/tutorials/install/).

## 创建Nginx配置

    在安装nginx之前，需要先创建rancher代理配置文件`/etc/nginx/conf.d/rancher.conf`。

    1. 复制粘贴以下文件到编辑器，并保存到 `/etc/nginx/conf.d/rancher.conf`.

        **NGIN示例配置:**

        ```conf
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
            server_name FQDN;
            ssl_certificate /certs/fullchain.pem;
            ssl_certificate_key /certs/privkey.pem;

            location / {
                proxy_set_header Host $host;
                proxy_set_header X-Forwarded-Proto $scheme;
                proxy_set_header X-Forwarded-Port $server_port;
                proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
                proxy_pass http://rancher;
                proxy_http_version 1.1;
                proxy_set_header Upgrade $http_upgrade;
                proxy_set_header Connection $connection_upgrade;
                # This allows the ability for the execute shell window to remain open for up to 15  minutes. Without this parameter, the default is 1 minute and will automatically close.
                proxy_read_timeout 900s;
            }
        }
        server {
            listen 80;
            server_name FQDN;
            return 301 https://$server_name$request_uri;
        }
        ```

    2. 在`/etc/nginx/conf.d/rancher.conf`中, 替换 `IP_NODE_1`, `IP_NODE_2`,  `IP_NODE_3` 为需要添加到集群的Linux主机的IP；

    3. 在`/etc/nginx/conf.d/rancher.conf`中, 替换`FQDN`为你设置用来登录rancher的域名；

    4. 在`/etc/nginx/conf.d/rancher.conf`中, 替换`/certs/fullchain.pem`为证书的路径；

    5. 在`/etc/nginx/conf.d/rancher.conf`中, 替换`/certs/privkey.pem`为证书密钥的路径；

## 运行NGINX

    - 重新加载或者重启NGINX

        ````bash
        # Reload NGINX
        nginx -s reload
        # Restart NGINX
        # Depending on your Linux distribution
        service nginx restart
        systemctl restart nginx
        ````

## 访问Rancher UI

    安装成功后，通过`https://FQDN`来访问RANCHER UI
