---
title: 1 - 概述
description: RancherOS is a simplified Linux distribution built from containers, for containers. These documents describe how to install and use RancherOS.
weight: 1
---

在生产中运行Docker，最小、最简单的方法之一是使用RancherOS。

RancherOS中的所有组件都是由Docker管理的容器运行，包括系统服务，如udev和syslog。因为RancherOS只包含运行Docker所需的服务，所以RancherOS比大多数传统操作系统小得多。通过删除不必要的库和服务，可以大大减少对安全补丁和其他维护的要求。因为使用Docker，用户通常会将所有必需的库打包到其容器中，所以宿主机系统不一定需要这些库文件。

RancherOS专为运行Docker而设计的另一种表现是它始终运行最新版本的Docker，允许用户使用最新的Docker功能和修复错误。

与其他极简主义的Linux发行版一样，RancherOS的启动速度非常快，通常在5-10秒内完成。启动Docker容器几乎是即时的，类似于启动任何其他进程。这种速度非常适合采用微服务和自动扩展的组织架构。

Docker是一个专为开发人员、系统管理员和DevOps设计的开源平台，它用于构建，传输和运行容器，使用简单而强大的CLI（命令行界面），您可以从Docker[用户指南](https://docs.docker.com/engine/userguide/)开始使用Docker 。

## 硬件要求

* 内存要求

平台 | RAM要求
---- | ----
Baremetal | 1280MB
VirtualBox | 1280MB
VMWare | 1280MB (rancheros.iso) <br> 2048MB (rancheros-vmware.iso)
GCE |  1280MB
AWS |  1.7GB

您可以通过自定义构建RancherOS来调整内存要求，请参阅[reduce-memory-requirements]({{< baseurl >}}/os/v1.x/en/installation/custom-builds/custom-rancheros-iso/#reduce-memory-requirements)

## RancherOS工作原理

RancherOS中的所有组件都是Docker容器，我们通过启动两个Docker实例来实现这一目标。一个是我们所说的`System Docker`，它是系统上的第一个进程(PID=1)。所有其他系统服务，如`ntpd，syslog和console`，都在Docker容器中运行。`System Docker`替代了传统的init系统，如systemd，可用于启动[其他系统服务]({{< baseurl >}}/os/v1.x/en/installation/system-services/adding-system-services/)。

`System Docker`运行一个名为`Docker`的特殊容器，它是另一个Docker守护进程，负责管理用户的所有容器,我们称之为`User Docker`。用户从控制台启动的任何容器都将在这个Docker中运行。`User Docker`容器与`System Docker`容器的相互隔离，确保正常用户命令不会影响系统服务。

![How it works]({{< baseurl >}}/img/os/rancheroshowitworks.png)

## 运行RancherOS

要了解更多关于安装RancherOS的信息，请跳转到[Quick Start Guide]({{< baseurl >}}/os/v1.x/cn/quick-start-guide/).

## 最新版本

请查看我们存储库中[README](https://github.com/rancher/os/blob/master/README.md)以获取最新版本。
