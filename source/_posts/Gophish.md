---
title: Gophish 配置教程
date: 2026-05-16 01:29:00
categories: 
  - RedTeam
tags:
  - 钓鱼邮件
---

# Gophish 配置教程

## 配置发件人

点击左侧 Sending Profiles -> New Profile

![](https://pic1.imgdb.cn/item/6a0771bc2990a55978ab61b7.png)

字段含义如下：

- 1. Name：给这个配置起个易记的名字

- 2. Interface Type：保持 SMTP 即可，GoPhish 目前只支持 SMTP

- 3. SMTP From：这是邮件客户端显示的 "发件人"，不会被 SMTP 服务器验证，你可以任意伪造

- 4. Host：邮箱服务器及端口

- 5. Username：填 SMTP 服务器的登录用户名

- 6. Password：填 SMTP 授权码

- 7. Ignore Certificate Errors：当 SMTP 服务器使用自签名证书或证书链不完整时勾选，避免 TLS 握手失败

- 8. Email Headers：用来给邮件增加自定义头

**发件人地址必须和你登录使用的 QQ 邮箱地址一致！！！**

**这里填的腾讯邮箱只是为了演示**

![](https://pic1.imgdb.cn/item/6a0774d22990a55978ab62ca.png)

## 导入目标用户

点击 Users & Groups -> New Group

字段含义如下：

- 1. First Name：收件人的名

- 2. Last Name：收件人的姓

- 3. Email：目标邮箱地址

- 4. Position：职位

![](https://pic1.imgdb.cn/item/6a0777272990a55978ab62e5.png)

## 制作诱饵

### 邮件模板

点击 Email Templates -> New Template

先在 Foxmail 等客户端设计好邮件，保存为 .eml 文件，再用 Gophish 的 Import Email 功能导入，可完美保留样式

字段含义如下：

- 1. Name：模板的内部名称，仅用于管理

- 2. Envelope Sender：返回路径信封发件人，会出现在邮件头 Return-Path 中，用于接收退信

- 3. Subject：邮件主题

- 4. Text：纯文本内容

- 5. HTML：HTML 格式邮件正文

- 6. Add Tracking Image：是否在邮件中嵌入一个透明跟踪图片，GoPhish 会记录其 "邮件已打开" 事件

**Envelope Sender 会覆盖掉 Sending Profile 里的设置，也必须和你发送的地址一致！！！**

![](https://pic1.imgdb.cn/item/6a0779e22990a55978ab62f7.png)

### 钓鱼页面

点击 Landing Pages -> New Page

可使用 Import Site 功能克隆目标网站。为提高成功率，务必勾选 Capture Submitted Data 和 Capture Passwords 来捕获凭证

![](https://pic1.imgdb.cn/item/6a077afd2990a55978ab6306.png)

## 攻击

点击 Campaigns -> New Campaign

![](https://pic1.imgdb.cn/item/6a07800a2990a55978ab7f52.png)

![](https://pic1.imgdb.cn/item/6a077ff92990a55978ab7f4e.png)