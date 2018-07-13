---
title: Rancher HA安装
weight: 5
---

本章节将介绍在开发和生产环境中通过4层负载均衡和7层负载均衡来部署Rancher HA。

- [四层负载均衡HA部署(TCP-L4)]({{< baseurl >}}/rancher/v2.x/cn/installation/server-installation/ha-install-external-lb/tcp-l4/)

- [七层负载均衡HA部署(HTTPS-L47]({{< baseurl >}}/rancher/v2.x/cn/installation/server-installation/ha-install-external-lb/https-l7/)

## 4层代理与7层代理的区别

| OSI中的层  | 功能                                     | TCP/IP协议族                                                 |
| ---------- | ---------------------------------------- | ------------------------------------------------------------ |
| 应用层     | 文件传输，电子邮件，文件服务，虚拟终端   | TFTP，HTTP，HTTPS，SNMP，FTP，SMTP，DNS，RIP，Telnet                |
| 表示层     | 数据格式化，代码转换，数据加密           | ASCII、ASN.1、JPEG、MPEG                                     |
| 会话层     | 解除或建立与别的接点的联系               | NetBIOS、ZIP                                                 |
| 传输层     | 位**数据段**提供端对端的接口             | TCP，UDP，SPX                                                |
| 网络层     | 为**数据包**选择路由，拥塞控制、网际互连 | IP，ICMP，OSPF，BGP，IGMP，ARP，RARP，IPX、RIP、OSPF         |
| 数据链路层  | 传输有地址的**帧**以及错误检测功能       | SLIP，CSLIP，PPP，MTU，ARP，RARP，SDLC、HDLC、PPP、STP、帧中继 |
| 物理层     | 以**二进制**数据形式在物理媒体上传输数据 | ISO2110，IEEE802，IEEE802.2，EIA/TIA RS-232、EIA/TIA RS-449、V.35、RJ-45 |

如图，平时我们说的7层和4层其实就是OSI网络模型中的第四层和第七层。4层代理中主要是TCP协议代理，TCP代理主要基于IP+端口来通信。7层代理中主要是http协议, http协议主要基于URL来通信。