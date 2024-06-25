---
title: git通过ssh连接github
comments: false
date: 2024-05-27 20:01:37
tags:
    - Git
categories:
    - Tool
    - Git
keywords:
description:
top_img:
cover: https://cnlicm-blog-image.oss-cn-shenzhen.aliyuncs.com/img/20240527200328.png
abbrlink:
---

## Git简介

&emsp;&emsp;Git是一个分布式版本控制系统，用于跟踪文件的变化并协调多个人在同一个项目上的工作。它最初由Linus Torvalds创建，用于管理Linux内核开发，现已成为许多软件开发团队和项目的标准工具之一。

<center>git版本管理流程</center>

![](https://cnlicm-blog-image.oss-cn-shenzhen.aliyuncs.com/img/20240527200903.png)

## SSH简介

&emsp;&emsp;SSH（Secure Shell）是一种网络协议，用于在不安全的网络上安全地进行远程登录和执行命令。它提供了加密的通信机制，可以确保在客户端和服务器之间传输的数据是安全的，不会被窃听或篡改。

    SSH的主要特点包括：

- 加密通信：SSH使用加密算法对通信数据进行加密，防止数据被第三方窃听或篡改。
- 认证机制：SSH支持多种认证方式，包括基于密码的认证、基于公钥的认证以及基于身份证书的认证，以确保用户身份的安全性。
- 安全性：SSH设计时考虑了各种安全问题，包括密码猜测、中间人攻击等，并提供了相应的安全机制和措施来保护系统免受这些攻击。
- 远程访问：SSH允许用户通过网络远程登录到其他计算机，并在远程计算机上执行命令或访问文件。
- 文件传输：除了远程登录，SSH还支持安全地传输文件，通过SCP（Secure Copy Protocol）或SFTP（SSH File Transfer Protocol）等方式。

{% note info%}
SSH已经成为了许多操作系统和网络设备的标准远程登录协议，广泛用于系统管理、远程开发、数据传输等场景。
{% endnote %}

## Git配置

### 配置用户名和邮箱

```Bash
git config --global user.name "username"
git config --global user.email "email"
```

### 生成SSH Key

```Bash
# 你的Github绑定的邮箱
ssh-keygen -t rsa -C "email"
```

### 获取SSH Key

```Bash
# 打开目录 %userprofile%/.ssh
cd %userprofile%/.ssh

# 复制 id_rsa.pub 文件内容
cat id_rsa.pub
```

### Github配置SSH

1. 进入[Github](github.com) > 点击头像 > 点击Settings
1. 点击`SSH and GPG keys` > 点击`New SSH Key`
    ![](https://cnlicm-blog-image.oss-cn-shenzhen.aliyuncs.com/img/20240527202143.png)
1. 输入公钥 > 点击`Add SSH Key`
    ![](https://cnlicm-blog-image.oss-cn-shenzhen.aliyuncs.com/img/20240527202249.png)

### 检测是否配置成功

```Bash
ssh -T git@github.com
```

<center>显示如下，表示配置成功</center>

![](https://cnlicm-blog-image.oss-cn-shenzhen.aliyuncs.com/img/20240527202613.png)