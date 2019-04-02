---
title: 1 - 准备主机和负载均衡器
weight: 1
aliases:
---
根据我们的要求需提供三个离线环境的Linux主机，以便保证HA配置中启动Rancher，节点需要有权限访问私有镜像仓库和私有git代码存储库。

## 主机需求

访问[节点需求]({{< baseurl >}}/rancher/v2.x/cn/install-prepare/requirements/)和[端口需求]({{< baseurl >}}/rancher/v2.x/cn/install-prepare/references/)了解HA安装的节点要求

## 推荐架构

- Rancher的DNS应解析为第4层负载均衡器
- Load Balancer应将端口TCP/80和TCP/443转发到Kubernetes集群中的所有节点。
- Ingress控制器将HTTP重定向到HTTPS并终止端口`TCP/443`上的SSL/TLS请求。

![Rancher HA]({{< baseurl >}}/img/rancher/ha/rancher2ha.svg)

## 二、配置负载均衡器(以NGINX为例)

我们将使用NGINX作为第4层负载均衡器(TCP)。NGINX会将所有连接转发到你的Rancher节点之一。

>**注意:** 在此配置中，负载平衡器位于Rancher节点的前面，负载均衡器可以是任意能够运行NGINX的主机。`不要使用任意一个Rancher节点作为负载均衡器节点`,会出现端口冲突。

### 1、安装nginx

首先在负载均衡器主机上安装NGINX，NGINX具有适用于所有已知操作系统的软件包。有关安装NGINX的帮助，请查阅其安装文档[nginx安装文档](https://www.nginx.com/resources/wiki/start/topics/tutorials/install/).

### 2、创建NGINX配置

安装NGINX后，你需要使用节点的IP地址更新NGINX配置文件`nginx.conf`。

1. 复制下面的代码到文本编辑器，保存为`nginx.conf`。
2. 在`nginx.conf`配置中, 替换`IP_NODE_1`、`IP_NODE_2`、`IP_NODE_3` 为你主机真实的IP地址。

    **NGINX配置示例:**

    ```bash
    worker_processes 4;
    worker_rlimit_nofile 40000;

    events {
        worker_connections 8192;
    }

    stream {
        upstream rancher_servers_http {
            least_conn;
            server <IP_NODE_1>:80 max_fails=3 fail_timeout=5s;
            server <IP_NODE_2>:80 max_fails=3 fail_timeout=5s;
            server <IP_NODE_3>:80 max_fails=3 fail_timeout=5s;
        }
        server {
            listen     80;
            proxy_pass rancher_servers_http;
        }

        upstream rancher_servers_https {
            least_conn;
            server <IP_NODE_1>:443 max_fails=3 fail_timeout=5s;
            server <IP_NODE_2>:443 max_fails=3 fail_timeout=5s;
            server <IP_NODE_3>:443 max_fails=3 fail_timeout=5s;
        }
        server {
            listen     443;
            proxy_pass rancher_servers_https;
        }
    }
    ```

3. 保存 `nginx.conf` ，并复制`nginx.conf`到负载均衡器节点的`/etc/nginx/nginx.conf`路径下。
4. 重新加载nginx配置

    ```bash
    nginx -s reload
    ```

### 3、可选 - 以容器运行nginx服务

我们可以以容器的形式运行nginx服务，而不需要把它安装在宿主机上。将编辑好的NGINX示例配置文件保存到/etc/nginx.conf，并运行以下命令来启动NGINX容器:

```bash
docker run -d --restart=unless-stopped \
  -p 80:80 -p 443:443 \
  -v /etc/nginx.conf:/etc/nginx/nginx.conf \
  nginx:1.14
```

## 三、FAQ和故障排除

[FAQ]({{< baseurl >}}/rancher/v2.x/cn/faq/)中整理了常见的问题与解决方法。