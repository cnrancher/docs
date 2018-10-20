---
title: 2 - RKE HA安装
weight: 2
---
>**重要提示:**
>RKE HA安装仅支持Rancher v2.0.8之前的版本，Rancher v2.1.0之后的版本使用[helm安装Rancher]({{< baseurl >}}/rancher/v2.x/cn/installation/server-installation/ha-install/helm-rancher/)。

本章节将介绍在开发和生产环境中通过4层负载均衡和7层负载均衡来部署Rancher HA。

## 四层代理与七层代理的区别

| OSI中的层     | 功能                                     | TCP/IP协议族                                                 |
| ------------ | ---------------------------------------- | ------------------------------------------------------------ |
| 应用层        | 文件传输，电子邮件，文件服务，虚拟终端   | TFTP，HTTP，HTTPS，SNMP，FTP，SMTP，DNS，RIP，Telnet                |
| 表示层        | 数据格式化，代码转换，数据加密           | ASCII、ASN.1、JPEG、MPEG                                     |
| 会话层        | 解除或建立与别的接点的联系               | NetBIOS、ZIP                                                 |
| 传输层        | 位**数据段**提供端对端的接口             | TCP，UDP，SPX                                                |
| 网络层        | 为**数据包**选择路由，拥塞控制、网际互连 | IP，ICMP，OSPF，BGP，IGMP，ARP，RARP，IPX、RIP、OSPF         |
| 数据链路层     | 传输有地址的**帧**以及错误检测功能       | SLIP，CSLIP，PPP，MTU，ARP，RARP，SDLC、HDLC、PPP、STP、帧中继 |
| 物理层        | 以**二进制**数据形式在物理媒体上传输数据 | ISO2110，IEEE802，IEEE802.2，EIA/TIA RS-232、EIA/TIA RS-449、V.35、RJ-45 |

如图，平时我们说的7层和4层其实就是OSI网络模型中的第四层和第七层。4层代理中主要是TCP协议代理，TCP代理主要基于IP+端口来通信。7层代理中主要是http协议, http协议主要基于URL来通信。

通过外部负载均衡来实现Rancher HA主要有两种类型:

### 外部四层负载均衡 + Rancher HA部署

- [四层负载均衡HA部署(TCP-L4)]({{< baseurl >}}/rancher/v2.x/cn/installation/server-installation/ha-install-external-lb/tcp-l4/)

所谓的四层负载均衡，就是让负载均衡器工作在传输层上，在这个层面上主要以TCP协议来传输数据。而我们平时给网站配置的ssl证书需要运行在http协议，所以四层负载均衡模式下将无法在负载均衡层面配置ssl证书。

因为Rancher2.x运行必须通过ssl进行加密，所以必须要配置ssl证书。负载均衡层无法配置ssl，我们可以把ssl证书放在后端。在Rancher推荐的部署架构中，Rancher server是运行在ingress后面，因此，ssl证书可以通过RKE的配置文件把证书信息传递给ingress。如果说架构中没有ingress，那么就需要把证书直接配置到Rancher server上，类似于[单节点安装]({{< baseurl >}}/rancher/v2.x/cn/installation/server-installation/single-node-install/)中，直接把证书映射给Rancher。

### 外部七层负载均衡 + Rancher HA部署

- [七层负载均衡HA部署(HTTPS-L7]({{< baseurl >}}/rancher/v2.x/cn/installation/server-installation/ha-install-external-lb/https-l7/)

七层负载运行在应用层，在这个层面上主要以http协议来传输数据，比如我们常用的WEB浏览器。我们平时用的HTTPS(也就是HTTP over TLS),就是在HTTP基础上利用SSL/TLS来加密。所以，以外部七层负载均衡器作为代理层，后端(ingress或者Rancher server)就不需要配置ssl证书，外部七层负载均衡器将作为ssl的终点。

默认情况，如果以`docker run -p 80:80 -p 443:443`运行一个Rancher server容器，访问80端口时会自动重定向到443端口上，也就是Rancher必须要求运行在`HTTPS`协议上。当外部七层负载均衡配置ssl证书后，后端的服务可以不用配置ssl证书，让它运行在`HTTP`协议上。 

对应此架构，在Rancher的开发设计中预制了一个功能开关，可以通过外部负载均衡器传递一个`X-Forwarded-Proto`参数给Rancher,这样Rancher容器就能知道外部已启用HTTPS，就会禁用Rancher server容器自身`80→443`的重定向功能,具体配置请查阅[七层负载均衡HA部署(HTTPS-L7]({{< baseurl >}}/rancher/v2.x/cn/installation/server-installation/ha-install-external-lb/https-l7/)。