---
title: 实战——Tomcat 站点渗透
date: 2026-05-25 07:15:00
categories: 
  - 渗透测试
tags:
  - 任意文件读取
  - 弱口令
  - RCE
---

# 实战——Tomcat 站点渗透

## 前言

Apache Tomcat 是一个开源的轻量级 Java Web 服务器和 Servlet 容器

它能加载、管理并执行 `.war` 包中的 Java Web 应用

当用户通过浏览器访问时，Tomcat 负责：

- 接收 HTTP 请求，并将其封装成 Java 对象（`HttpServletRequest`）

- 定位并调用对应的 Servlet（`doGet`、`doPost` 等方法）

- 执行 JSP 文件：将 `.jsp` 文件编译成 Servlet 后再执行

- 返回 HTTP 响应给客户端

## 弱口令

常见的弱口令如下：

```
tomcat:tomcat
admin:123456
tomcat:123456
```

登录后页面长这个样子

![](https://pic1.imgdb.cn/item/6a13b1dafe89374d2095149f.png)

## AJP 文件包含（CVE-2020-1938）

Apache Tomcat 作为一个 Java Web 服务器，主要通过两种 Connector 组件与外界通信：

- HTTP Connector：默认监听 8080 端口，用于处理来自客户端浏览器的 HTTP 请求

- AJP Connector：默认监听 8009 端口，用于处理来自前端 Web 服务器的 AJP 请求。其设计初衷是提高静态资源处理性能

问题就在于，这个默认开启的 AJP 服务（端口 8009），其协议实现上存在一处文件包含缺陷

攻击者向目标 Tomcat 服务器的 AJP 端口（默认8009）发起连接，并发送一个精心构造的恶意 AJP 请求

参数注入：在这个恶意请求中，攻击者会注入三个关键的请求属性：

- `javax.servlet.include.request_uri` 原始客户端的完整请求 URI（`/app/servletA/extra/info.jsp`）

- `javax.servlet.include.path_info` 原始请求中匹配 Servlet 的路径部分（`/servletA`）

- `javax.servlet.include.servlet_path` 原始请求中属于路径信息的部分（`/extra/info.jsp`）

绕过访问控制：通过操纵这些属性，攻击者能够绕过原本的访问控制机制，让 Tomcat 服务器将攻击者指定的文件当作一个 JSP 文件来处理

攻击脚本：https://github.com/00theway/Ghostcat-CNVD-2020-10487#

读取 `WEB-INF/web.xml`，可据此绘制攻击路线图

![](https://pic1.imgdb.cn/item/6a13b5eefe89374d20951569.png)

## War 包上传 GetShell

在这里上传部署，但是基本上已经不行了

![](https://pic1.imgdb.cn/item/6a13be854a8f57a65a3bdaa2.png)