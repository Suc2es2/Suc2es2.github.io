---
title: VulnHub Homeless：1
date: 2026-04-20 23:19:00
categories: 
  - VulnHub
tags:
  - 弱口令
  - 文件上传
  - 代码审计
  - RCE
  - Python 升级为交互式 Shell
  - 定时任务
  - 短标签绕过长度限制
  - MD5 弱比较绕过
---

# Homeless: 1

项目地址：https://download.vulnhub.com/homeless/Homeless.zip

是一个打包好的镜像文件

**注意：导入 vmx 时一定要选择我已移动，否则没网**

使用 Nmap 扫描出开放服务及操作系统版本

![](https://pic1.imgdb.cn/item/69e644fa69eaadff3e5c14f4.png)

## CTF

### index（UA 头爆破）

扫描后台没东西，看看 `robots.txt` 发现提示字典 rockyou

![](https://pic1.imgdb.cn/item/69e647cb69eaadff3e5c155d.png)

访问 80 端口看到有回显内容，BurpSuite 检查确认

![](https://pic1.imgdb.cn/item/69e64cde69eaadff3e5c1772.png)

结合给的爆破字典提示，尝试爆破这个 UA 头内容

爆破出关键字 `cyberdog`，让我们访问一个路径

![](https://pic1.imgdb.cn/item/69e64dc469eaadff3e5c188d.png)

拼接到浏览器访问，是一个文件上传页面

![](https://pic1.imgdb.cn/item/69e64dfe69eaadff3e5c1891.png)

## 文件上传

### Upload Me（短标签绕过长度限制）

直接传反弹 Shell 回显太大了，推测做了大小检测，且应该很小

![](https://pic1.imgdb.cn/item/69e64e7569eaadff3e5c4e97.png)

利用 PHP 短标签 `<?=` 配合双引号包裹的两字符命令，构造 8 字节 的代码执行载荷

```php
<?='ls';
```

上传成功

![](https://pic1.imgdb.cn/item/69e64f3169eaadff3e5c4ea4.png)

访问路径拿到回显，明显发现一个 txt 文件

![](https://pic1.imgdb.cn/item/69e651ac69eaadff3e5c4eb7.png)

直接访问拿到新提示

![](https://pic1.imgdb.cn/item/69e651e869eaadff3e5c4eba.png)

访问过去是一个新的登录界面

![](https://pic1.imgdb.cn/item/69e6520d69eaadff3e5c4ebc.png)

点击提示拿到网页源码

![](https://pic1.imgdb.cn/item/69e6523f69eaadff3e5c4ebe.png)

## 代码审计

### Sign In（MD5 弱比较绕过）

关键代码如下

![](https://pic1.imgdb.cn/item/69e652c469eaadff3e5c4ec4.png)

PHP 中形如 `0e123...` 的字符串会被解释为科学计数法 0，当两个不同的字符串经 MD5 后均以此格式开头时，`==` 比较将判定为 True

使用下面的工具快速生成 MD5 相同的文件（因为我们需要三个）

`https://github.com/grevutiu-gabriel/python-md5-collision/tree/master`

但其原始二进制数据无法直接通过 HTTP 表单传输

通过 Python 脚本将碰撞数据 URL 编码 后发送，成功绕过登录验证，进入命令执行面板

![](https://pic1.imgdb.cn/item/69e6562369eaadff3e5c504b.png)

## RCE

### admin

测试发现没有任何限制，可以畅快的执行任何命令

![](https://pic1.imgdb.cn/item/69e6563569eaadff3e5c504c.png)

直接弹 Shell

![](https://pic1.imgdb.cn/item/69e6587669eaadff3e5c50f4.png)

## 提权

### Python 升级为交互式 Shell

```python
python -c 'import pty;pty.spawn("/bin/bash")'
```

### SSH 爆破

打到最后还得爆破这个用户，倒不如一开始就爆破呢

![](https://pic1.imgdb.cn/item/69e6584869eaadff3e5c50f2.png)

### 定时任务

查看提示

![](https://pic1.imgdb.cn/item/69e6580c69eaadff3e5c50ef.png)

发现这个文件，root 用户是有执行权的

![](https://pic1.imgdb.cn/item/69e6582a69eaadff3e5c50f0.png)

我们还有一封邮件待查看，看看是什么

![](https://pic1.imgdb.cn/item/69e658a669eaadff3e5c50f6.png)

应该是计划任务每分钟执行一次 `homeless.py` 脚本，及报错信息

downfall 有写权限，所以可以打定时任务提权

![](https://pic1.imgdb.cn/item/69e658f569eaadff3e5c50fa.png)

提权成功

![](https://pic1.imgdb.cn/item/69e6590269eaadff3e5c50fb.png)
