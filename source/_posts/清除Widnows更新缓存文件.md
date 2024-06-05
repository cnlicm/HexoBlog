---
title: 清除Widnows更新缓存文件
comments: false
date: 2024-06-01 11:53:48
tags:
    - WindowsUpdate
categories:
    - 工具
    - WindowsUpdate
keywords:
description:
top_img:
cover: https://cnlicm-blog-image.oss-cn-shenzhen.aliyuncs.com/img/20240601115931.png
abbrlink:
---

## 通过[命令提示符]删除Windows更新缓存文件

1. 使用`Windows + R`快捷键打开[运行]对话框，输入`cmd`，然后按`Ctrl + Shift + Enter`以管理员权限打开[命令提示符]
1. 执行以下命令停止[Windows Update]服务
    
    ```Bash
    net stop wuauserv
    ```
1. 执行以下命令进入到`SoftwareDistribution`目录

    ```Bash
    cd %Windir%\SoftwareDistribution
    ```
1. 执行以下命令，强制删除`Download`文件夹及其子文件夹中的所有文件

    ```Bash
    del /f /s /q Download
    ```
1. 执行以下命令，强制删除`DataStore`文件夹及其子文件夹中的所有文件

    ```Bash
    del /f /s /q DataStore
    ```
1. 执行以下命令重新启动[Windows Update]服务

    ```Bash
    net start wuauserv
    ```
<img src="https://cnlicm-blog-image.oss-cn-shenzhen.aliyuncs.com/img/20240601120301.png"/>
