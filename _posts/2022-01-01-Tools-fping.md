---
title: Tools-fping
author: AmtuOrRF
categories: [tools]
tags: [tools]
---

fping 是一个类似 ping 的程序，它使用 Internet 控制消息协议 (ICMP) 回显请求来确定目标主机是否正在响应。fping 与 ping 的不同之处在于您可以在命令行上指定任意数量的目标，或者指定一个包含要 ping 的目标列表的文件。fping 不会在超时或回复之前发送到一个目标，而是发送一个 ping 数据包并以循环方式移动到下一个目标。

向网络主机发送 ICMP ECHO_REQUEST 数据包

Help
```shell
root@kali:~# fping -h
Usage: fping [options] [targets...]

Probing options:
   -4, --ipv4 仅 ping IPv4 地址
   -6, --ipv6 仅 ping IPv6 地址
   -b, --size=BYTES 要发送的 ping 数据量，以字节为单位（默认值：56）
   -B, --backoff=N 将指数退避因子设置为 N（默认值：1.5）
   -c, --count=N 计数模式：向每个目标发送 N 个 ping
   -f, --file=FILE 从文件中读取目标列表（ - 表示标准输入）
   -g, --generate 生成目标列表（仅当未指定 -f 时）
                      （在目标列表中给出开始和结束 IP，或 CIDR 地址）
                      （例如 fping -g 192.168.1.0 192.168.1.255 或 fping -g 192.168.1.0/24）
   -H, --ttl=N 设置 IP TTL 值（Time To Live hops）
   -I, --iface=IFACE 绑定到特定接口
   -l, --loop 循环模式：永远发送 ping
   -m, --all 使用提供的主机名的所有 IP（例如 IPv4 和 IPv6），与 -A 一起使用
   -M, --dontfrag 设置不分片标志
   -O, --tos=N 在 ICMP 数据包上设置服务类型 (tos) 标志
   -p, --period=Ping 数据包到一个目标之间的 MSEC 间隔（以毫秒为单位）
                      （在循环和计数模式下，默认值：1000 ms）
   -r, --retry=N 重试次数（默认：3）
   -R, --random 随机数据包数据（用于阻止链接数据压缩）
   -S, --src=IP 设置源地址
   -t, --timeout=MSEC 单个目标初始超时（默认值：500 毫秒，
                      除了 -l/-c/-C，它的 -p 周期长达 2000 毫秒）

输出选项：
   -a, --alive 显示活动的目标
   -A, --addr 按地址显示目标
   -C，--vcount=N 同-c，以详细格式报告结果
   -D, --timestamp 在每个输出行之前打印时间戳
   -e, --elapsed 显示返回数据包经过的时间
   -i, --interval=发送 ping 数据包之间的 MSEC 间隔（默认值：10 毫秒）
   -n, --name 按名称显示目标（-d 等效）
   -N, --netdata 输出与 netdata 兼容（需要 -l -Q）
   -o, --outage 显示累计中断时间（丢包*包间隔）
   -q, --quiet quiet（不显示每个目标/每个 ping 的结果）
   -Q, --squiet=SECS 与 -q 相同，但每 n 秒显示一次摘要
   -s, --stats 打印最终统计数据
   -u, --unreach 显示无法访问的目标
   -v, --version 显示版本
   -x, --reachable=N 显示 >=N 主机是否可达****
```

## 使用示例

`-g` 扫描指定网段，将IP存活信息重定向到show.txt
```bash
fping -g 172.16.1.1/24 > show.txt
```
能ping通的 和 ping不通的 都显示
```shell
# head show.txt 
172.16.1.2 is alive
172.16.1.3 is alive
172.16.1.7 is alive
172.16.1.50 is alive
172.16.1.52 is alive
172.16.1.99 is alive
172.16.1.200 is alive
172.16.1.254 is alive
172.16.1.1 is unreachable
172.16.1.4 is unreachable
...
```

`-g -a` 扫描指定网段，将IP存活的重定向到show.txt
```bash
fping -g -a 172.16.1.1/24 > show.txt
```
只输出ping的通的ip
```shell
# head show.txt 
172.16.1.2
172.16.1.3
172.16.1.7
172.16.1.50
172.16.1.52
172.16.1.59
172.16.1.99
172.16.1.200
172.16.1.254
```

常用的就这写，你也可以继续扩展