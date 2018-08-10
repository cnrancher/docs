---
title: 404 - default backend
weight: 30
---

您需要下载kubectl命令行工具来调试此错误。kubectl命令行工具安装方法，请查阅See [kubectl安装与设置](https://kubernetes.io/docs/tasks/tools/install-kubectl/)。

当修改了 `rancher-cluster.yml`配置文件后, 必须运行`rke remove --config rancher-cluster.yml`以清理节点，确保不会与先前的配置发生冲突。

### 可能的原因

1. Nginx ingress无法为`rancher-cluster.yml`配置的应用提供代理服务

    这应该是您为访问Rancher的FQDN配置错误，您可以通过运行以下命令来检查它是否已正确配置：

    ```bash
    kubectl --kubeconfig kube_config_rancher-cluster.yml get ingress -n cattle-system -o wide
    ```

    检查`HOSTS`是否与配置文件的`<FQDN>`相同，以及列出的`ADDRESS`是否与配置文件配置的主机IP相同

    通过以下命令查看nginx ingress日志：

    ```bash
    kubectl --kubeconfig kube_config_rancher-cluster.yml logs -l app=ingress-nginx -n ingress-nginx
    ```

2. `x509: certificate is valid for fqdn, not your_configured_fqdn`

    证书中包含的域名与实际的域名不匹配；

3. `Port 80 is already in use. Please check the flag --http-port`

    There is a process on the node occupying port 80, this port is needed for the nginx ingress controller to route requests to Rancher. You can find the process by running the command: `netstat -plant | grep \:80`.

    Stop/kill the process and redeploy.

4. `unexpected error creating pem file: no valid PEM formatted block found`

    The base64 encoded string configured in the template is not valid. Please check if you can decode the configured string using `base64 -D STRING`, this should return the same output as the content of the file you used to generate the string. If this is correct, please check if the base64 encoded string is placed directly after the key, without any newlines before, in between or after. (For example: `tls.crt: LS01..`)
