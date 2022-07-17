---
title: ssh-no-matching-key-exchange-method-found
author: AmtuOrRF
categories: [Misc]
tags: [Misc]
---
> 2022年4月19日,星期二

解决ssh连接时出现以下类似的错误--
```bash
root@CLw0rm:~# ssh root@172.16.1.55
Unable to negotiate with 172.16.1.55 port 22: no matching key exchange method found. Their offer: diffie-hellman-group-exchange-sha1,diffie-hellman-group14-sha1,diffie-hellman-group1-sha1
```
### 方法1:
在当前用户的.ssh目录下新建config文件,只对当前用户生效;
```shell
$ vim  ~/.ssh/config
```
添加以下内容:
```shell
Host *
KexAlgorithms +diffie-hellman-group1-sha1
```

### 方法1:
修改/etc/ssh/ssh_config文件;
```shell
$ vim /etc/ssh/ssh_config
```
在文件ssh配置文件末尾添加以下内容,全不用户生效.
```text
KexAlgorithms +diffie-hellman-group1-sha1
```