---
title: hexo部署记录
comments: false
date: 2024-05-12 17:17:14
tags:
    - Hexo
categories:
    - Hexo
keywords:
description:
top_img:
cover:
abbrlink:
---

## Hexo部署记录

{% note info %}

当前部署方式均来源于网络，当前记录只作为记录参考，自行食用

{% endnote %}

### 自定义域名

- 访问[Vercel](https://vercel.com/)，点击`Sign up`通过`Github`注册账号
- 注册完成后，点击`New Project`，选择`Import Git Repository`
- 在`Import Git Repository`中，选择`hexo`仓库，点击`Import`
- 在`Configure Project`页，填写相关信息点击`Deploy`等待完成配置
- 在控制面板点击项目，即可查看网站
- 项目配置完成后，在`Settings`中，找到`Domains`，点击`Add a domain`
- 输入自定义域名，点击`Add`等待完成配置
- 根据提示，在`DNS`中添加域名解析解析，等待完成配置
![自定义域名配置](http://static.bootree.cn/img/20240512173735.png)

### 自定义OSS存储图片

- 阿里云购买对象存储服务，创建存储空间
    ![OSS存储配置](http://static.bootree.cn/img/20240512180353.png)
- 进入`对象存储OSS`控制台，点击`Bucket列表`
- 创建`Bucket`，填写基本信息（其中`读写权限改为公共读`）
- 点击头像选择`访问控制`，进入`RAM访问控制`
- 点击`用户`，输入基本信息（勾选`OpenAPI`调用访问 启用AccessKey ID和AccessKey Secret）
- 用户首页点击`添加权限`，选择添加权限
    ![添加权限](http://static.bootree.cn/img/20240512181202.png)
- 点击用户名进入`用户详情页`，点击`创建AccessKey`得到`AccessKey ID和AccessKey Secret`
- 访问[PicGo](https://github.com/Molunerfinn/PicGo/releases)，下载并安装PicGo
- 进入`PicGo`应用，进行图床设置
    ![图床设置](http://static.bootree.cn/img/20240512182037.png)
- 可进行测试上传，在阿里云对象存储OSS控制台可以看到上传的图片，点击可看到上传的图片信息，在浏览器访问图片链接会触发下载，解决方案是需要配置自定义域名 [常见问题](https://help.aliyun.com/zh/oss/user-guide/how-to-ensure-an-object-is-previewed-when-you-access-the-object?spm=a2c4g.11186623.2.19.19681a214ISjKj)
- 进入`Bucket`详情页，侧边栏选择`Bucket配置`下的`域名管理`
- 输入绑定域名例如`static.bootree.cn`，会自动生成域名解析记录
    ![阿里云域名解析记录配置](http://static.bootree.cn/img/20240512191034.png)
- 通过[长期URL](https://help.aliyun.com/zh/oss/user-guide/map-custom-domain-names-5?spm=a2c4g.11186623.0.i2#4b2d958079f12)访问图片
    ![图片](http://static.bootree.cn/img/20240512191755.png)