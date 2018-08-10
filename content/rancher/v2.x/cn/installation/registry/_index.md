---
title: 镜像仓库安装
weight: 2
---

Harbor 是一个企业级的 Docker Registry，可以实现images的私有存储和日志统计权限控制等功能，并支持创建多项目(Harbor 提出的概念)，基于官方Registry实现。 通过地址：https://github.com/vmware/harbor/releases可以下载最新的版本。官方提供了两种版本：在线版和离线版。

- 在线安装：安装程序从Docker镜像仓库下载Harbour相关映像。因此，安装程序的尺寸非常小。
- 离线安装：主机没有Internet连接时使用此安装程序镜像安装。安装程序包含所有镜像，因此压缩包较大。
- OVA安装程序：当用户有vCenter环境时，使用此安装程序，在部署OVA后启动Harbor。详细信息请参考Harbour OVA安装指南

本文仅介绍使用在线安装和配置Harbor的步骤，离线安装过程几乎相同。
 
本文仅介绍全新安装Harbor，

