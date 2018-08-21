---
title: API Keys
weight: 1

---

## API Keys and User Authentication

如果要使用外部应用程序访问Rancher集群、项目或其他对象，可以使用Rancher API执行此操作。但是，在您的应用程序可以访问API之前，您必须为应用程序提供用于向Rancher进行身份验证的API密钥。您可以使用Rancher UI获取密钥。

> 使用Rancher CLI还需要API密钥[cli](../cli)。

API密钥由四个组件组成：

- **Endpoint:**

这是其他应用程序用于向Rancher API发送请求的地址。

- **Access Key:**

token的用户名

- **Secret Key:**

token的密码。对于提示您使用API身份验证的应用程序，通常会将两个密钥一起输入。

- **Bearer Token:**

token的用户名和密码连接在一起。将此字符串用于提示您输入一个验证字符串的应用程序。

## 创建API Key

1. Select **User Avatar** > **API & Keys** from the **User Settings** menu in the upper-right.

2. Click **Add Key**.

3. **Optional:** Enter a description for the API key and select an expiration period. We recommend setting an expiration date.

    The API key won't be valid after expiration. Shorter expiration periods are more secure.

4. Click **Create**.

    **步骤结果:** Your API Key is created. Your API **Endpoint**, **Access Key**, **Secret Key**, and **Bearer Token** are displayed.

    Use the **Bearer Token** to authenticate with Rancher CLI.

5. Copy the information displayed to a secure location. This information is only displayed once, so if you lose your key, you'll have to make a new one.

## What's Next?

- Enter your API key information into the application that will send requests to the Rancher API.
- Learn more about the Rancher endpoints and parameters by selecting **View in API** for an object in the Rancher UI.
- API keys are used for API calls and [Rancher CLI]({{< baseurl >}}/rancher/v2.x/en/cli).

## Deleting API Keys

If you need to revoke an API key, delete it. You should delete API keys:

- That may have been compromised.
- That have expired.

To delete an API, select the stale key and click **Delete**.
