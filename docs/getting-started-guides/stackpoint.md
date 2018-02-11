---
cn-approvers:
- tianshapjq
approvers:
- baldwinspc
title: 通过 Stackpoint.io 在多个云平台上运行 Kubernetes
---


* TOC
{:toc}



## 简介
StackPointCloud 是一个能够让 Kubernetes 随处运行的通用控制平面。通过 StackPointCloud，您可以使用基于 Web 的界面，通过3个步骤将 Kubernetes 集群部署到您选择的云提供商上，并能够管理集群。



## AWS

要在 AWS 上创建 Kubernetes 集群，您需要 AWS 的访问密钥 ID 和秘密访问密钥（Secret Access Key）。


### 选择一个提供商

通过 GitHub、Google 或者 Twitter 账号来登录 [stackpoint.io](https://stackpoint.io)。

点击 **+ADD A CLUSTER NOW**。

点击选择 Amazon Web Services (AWS)。


### 配置您的提供商

添加您从 AWS 获得的访问密钥 ID 和秘密访问密钥。选择您的默认 StackPointCloud SSH 密钥对，或者点击 **ADD SSH KEY** 来添加一个新的密钥对。

点击 **SUBMIT** 来提交验证信息。


### 配置您的集群

选择您希望包含在集群中的任何其他选项，然后单击 **SUBMIT** 创建集群。


### 运行集群

您能通过 [您的 stackpoint.io 面板](https://stackpoint.io/#/clusters) 来管理集群状态，也能进行挂起和删除操作。

对于如何在 AWS 上使用和管理 Kubernetes 集群，[请参阅 Kubernetes 文档](/docs/getting-started-guides/aws/)。





## GCE

要在 GCE 上创建 Kubernetes 集群，您需要 Google 提供的服务帐户 JSON 数据（Service Account JSON Data）。


### 选择一个提供商

通过 GitHub、Google 或者 Twitter 账号来登录 [stackpoint.io](https://stackpoint.io)。

点击 **+ADD A CLUSTER NOW**。

点击选择 Google Compute Engine (GCE)。


### 配置您的提供商

添加您从 Google 获得的服务帐户 JSON 数据。选择您的默认 StackPointCloud SSH 密钥对，或者点击 **ADD SSH KEY** 来添加一个新的密钥对。

点击 **SUBMIT** 来提交验证信息。


### 配置您的集群

选择您希望包含在集群中的任何其他选项，然后单击 **SUBMIT** 创建集群。


### 运行集群

您能通过 [您的 stackpoint.io 面板](https://stackpoint.io/#/clusters) 来管理集群状态，也能进行挂起和删除操作。

对于如何在 GCE 上使用和管理 Kubernetes 集群，[请参阅 Kubernetes 文档](/docs/getting-started-guides/gce/)。





## Google Kubernetes Engine


要在 Google Kubernetes Engine 上创建一个 Kubernetes 集群，你需要使用从 Google 获得的服务帐户 JSON 数据。


### 选择一个提供商

通过 GitHub、Google 或者 Twitter 账号来登录 [stackpoint.io](https://stackpoint.io)。

点击 **+ADD A CLUSTER NOW**。

点击选择 Google Kubernetes Engine。


### 配置您的提供商

添加您从 Google 获得的服务帐户 JSON 数据。选择您的默认 StackPointCloud SSH 密钥对，或者点击 **ADD SSH KEY** 来添加一个新的密钥对。

点击 **SUBMIT** 来提交验证信息。


### 配置您的集群

选择您希望包含在集群中的任何其他选项，然后单击 **SUBMIT** 创建集群。



### 运行集群

您能通过 [您的 stackpoint.io 面板](https://stackpoint.io/#/clusters) 来管理集群状态，也能进行挂起和删除操作。

对于如何在 Google Kubernetes Engine 上使用和管理 Kubernetes 集群，请参阅 [官方文档](/docs/home/)。


## DigitalOcean


如果想要在 DigitalOcean 上创建一个 Kubernetes 集群，您需要拥有一个 DigitalOcean 的 API token。


### 选择一个提供商

通过 GitHub、Google 或者 Twitter 账号来登录 [stackpoint.io](https://stackpoint.io)。

点击 **+ADD A CLUSTER NOW**。

点击选择 DigitalOcean。


### 配置您的提供商

添加您的 DigitalOcean API Token。选择您的默认 StackPointCloud SSH 密钥对，或者点击 **ADD SSH KEY** 来添加一个新的密钥对。

点击 **SUBMIT** 来提交验证信息。


### 配置您的集群

选择您希望包含在集群中的任何其他选项，然后单击 **SUBMIT** 创建集群。


### 运行集群

您能通过 [您的 stackpoint.io 面板](https://stackpoint.io/#/clusters) 来管理集群状态，也能进行挂起和删除操作。

对于如何在 DigitalOcean 上使用和管理 Kubernetes 集群，请参阅 [官方文档](/docs/home/)。





## Microsoft Azure


如果想要在 Microsoft Azure 上创建一个 Kubernetes 集群，您需要拥有一个 Azure Subscription ID，用户名/邮箱和密码。


### 选择一个提供商

通过 GitHub、Google 或者 Twitter 账号来登录 [stackpoint.io](https://stackpoint.io)。

点击 **+ADD A CLUSTER NOW**。

点击选择 Microsoft Azure。


### 配置您的提供商

添加您的 Azure Subscription ID、用户名/邮箱和密码。选择您的默认 StackPointCloud SSH 密钥对，或者点击 **ADD SSH KEY** 来添加一个新的密钥对。

点击 **SUBMIT** 来提交验证信息。


### 配置您的集群

选择您希望包含在集群中的任何其他选项，然后单击 **SUBMIT** 创建集群。


### 运行集群

您能通过 [您的 stackpoint.io 面板](https://stackpoint.io/#/clusters) 来管理集群状态，也能进行挂起和删除操作。

对于如何在 Azure 上使用和管理 Kubernetes 集群，请参阅 [Kubernetes 文档](/docs/getting-started-guides/azure/)。





## Packet


如果想要在 Packet 上创建一个 Kubernetes 集群，您需要拥有一个 Packet API 密钥。


### 选择一个提供商

通过 GitHub、Google 或者 Twitter 账号来登录 [stackpoint.io](https://stackpoint.io)。

点击 **+ADD A CLUSTER NOW**。

点击选择 Packet。


### 配置您的提供商

添加您的 Packet API 密钥。选择您的默认 StackPointCloud SSH 密钥对，或者点击 **ADD SSH KEY** 来添加一个新的密钥对。

点击 **SUBMIT** 来提交验证信息。


### 配置您的集群

选择您希望包含在集群中的任何其他选项，然后单击 **SUBMIT** 创建集群。


### 运行集群

您能通过 [您的 stackpoint.io 面板](https://stackpoint.io/#/clusters) 来管理集群状态，也能进行挂起和删除操作。

对于如何在 Packet 上使用和管理 Kubernetes 集群，请参阅 [官方文档](/docs/home/)。
