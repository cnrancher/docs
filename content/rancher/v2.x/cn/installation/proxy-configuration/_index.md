---
title: Rancher HTTP代理配置
weight: 4
---

如果你的环境需要通过代理才可以连接互联网，那么需要配置`http_proxy`

## 设置主机`http_proxy` 变量

### Ubuntu

1. 检查 `http_proxy` 变量是否已定义：

    ```bash
    echo $http_proxy
    ```
    如果变量为空，可以通过以下命令设置变量并将其存储在您帐户的环境中：

    ```bash
    echo "export http_proxy=http://<username>:<password>@<proxy url>:<proxy port>/" >> .profile
    ```

2. 注销并重新登录以激活您的更改

## 使用代理信息启动Rancher容器

确保http_proxy环境变量在Rancher的Docker容器内可见

```bash
HOST_PATH=/home/rancher/data
sudo docker run -d --restart=unless-stopped \
--volumes-from rancher-data -p 80:80 -p 443:443 \
-v /etc/localtime:/etc/localtime \
-v /$HOST_PATH/:/var/lib/rancher \
-e HTTP_PROXY=$http_proxy \
-e HTTPS_PROXY=$http_proxy \
-e http_proxy=$http_proxy \
-e https_proxy=$http_proxy \
-e NO_PROXY="localhost,127.0.0.1" \
-e no_proxy="localhost,127.0.0.1" \
rancher/rancher
```
