---
title: 日志审计

---


只有[管理员]({{< baseurl >}}/rancher/v1.x/cn/configuration/access-control/#管理员)用户有权限访问审计日志。审计日志在**系统管理**->**审计日志**。

Rancher的审计日志是不同事件类型的集合。

* 任何带有前缀`api`的事件是API的一次调用。事件类型将记录API操作，谁执行的操作以及API调用的方式(即通过UI，通过API密钥)。
* 任何没有带`api`前缀的事件都是Rancher Server做的事情。例如，在协调服务的容器期间，在实例创建时会产生一个`instance.create`事件。
