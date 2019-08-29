---
title: 5 - 配置Rancher系统chart
weight: 5
---

## 一、准备系统chart

该[系统chart](https://github.com/rancher/system-charts)库包含的所有功能，如moniotring，logging，报警和全局DNS等。为了能够在离线安装中使用这些功能，您需要将`system-charts`代码同步到内部git代码库中，以便Rancher可以访问。

## 二、配置系统chart

1. 登录Rancher

1. 浏览器访问`https://<your-rancher-server>/v3/catalogs/system-library`![æå¼](assets/system-charts-setting.png)

1. 单击右上角的“ **编辑”**，将**url**的值更新为内部Git代码库`system-charts`的地址。![æ´æ°](assets/system-charts-update.png)

   > **注意:** 在同步`system-charts`时，建议按照一比一的方式同步到内部GIT仓库，这样此处只需要修改`URL`地址。如果`system-charts`同步到本地GIT仓库后分支发生了变化，`branch`处需要修改为本地GIT仓库对应的分支。

1. **点击Show Request**

1. **点击Send Request**
