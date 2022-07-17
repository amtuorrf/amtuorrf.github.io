---
title: Tools-volatility
author: AmtuOrRF
categories: [tools]
tags: [tools]
---
## Description
> Volatility是一款非常强大的内存取证工具,它是由来自全世界的数百位知名安全专家合作开发的一套工具, 
> 可以用于windows,linux,mac osx,android等系统内存取证。Volatility是一款开源内存取证框架，
> 能够对导出的内存镜像进行分析，通过获取内核数据结构，使用插件获取内存的详细情况以及系统的运行状态.


## help
```bash
VolatilityFoundation 波动率框架 2.6
用法： Volatility - 内存取证分析平台。

选项：
  -h, --help              列出所有可用选项及其默认值。
                          默认值可以在配置文件中设置
                        (/etc/volatilityrc)
  --conf-file=/root/.volatilityrc
                          基于用户的配置文件
  -d, --debug             调试易变性
  --plugins=PLUGINS       要使用的附加插件目录（冒号分隔）
  --info                  打印所有注册对象的信息
  --cache-directory=/root/.cache/volatility
                          存放缓存文件的目录
  --cache                 使用缓存
  --tz=TZ 设置 (Olson)     时区以显示时间戳
                          使用 pytz（如果已安装）或 tzset
  -f filename，--profile=文件名
                          打开图像时使用的文件名
  --profile=Win7SP1x64    要加载的配置文件的名称（使用 --info 查看列表支持的配置文件）
  -l file:///root/vol/1.vmem, --location=file:///root/vol/1.vmem
                          从中加载地址空间的 URN 位置
  -w, --write             启用写支持
  --dtb=DTB DTB           地址
  --shift=SHIFT Mac KASLR 移位地址
  --output=text           以这种格式输出（支持是特定于模块的，请参阅下面的模块输出选项）
  --output-file=OUTPUT_FILE
                          在此文件中写入输出
  -v, --verbose           详细信息
  --physical_shift=PHYSICAL_SHIFT
                          Linux内核物理移位地址
  --virtual_shift=VIRTUAL_SHIFT
                          Linux内核虚拟移位地址
  -g KDBG, --kdbg=KDBG    指定一个 KDBG 虚拟地址（注意：对于 64 位Windows 8 及以上系统的地址KdCopyDataBlock)
  --force                 强制使用可疑配置文件
  --cookie=COOKIE         指定nt!ObHeaderCookie的地址（对仅限 Windows 10）
  -k KPCR, --kpcr=KPCR    指定具体的KPCR地址支持的插件命令：

amcache         打印 AmCache 信息
apihooks        检测进程和内核内存中的 API 挂钩
atom            打印会话和窗口站原子表
atomscan        原子表的池扫描器
auditpol        从 HKLM\SECURITY\Policy\PolAdtEv 打印出审计策略
bigpools        使用 BigPagePoolScanner 转储大页面池
bioskbd         从实模式内存中读取键盘缓冲区
cachedump       从内存中转储缓存的域哈希
                回调打印系统范围的通知例程
                剪贴板 提取 windows 剪贴板的内容
cmdline         显示进程命令行参数
cmdscan         通过扫描
                _COMMAND_HISTORY 提取命令历史记录
consoles        通过扫描
                _CONSOLE_INFORMATION 提取命令历史记录
crashinfo       转储崩溃转储信息
                用于 tagDESKTOP（台式机）的deskscan Poolscaner
devicetree      显示设备树
dlldump         从进程地址空间转储 DLL
dlllist         打印每个进程加载的 dll 列表
driverirp       驱动程序 IRP 挂钩检测
drivermodule    将驱动程序对象与内核模块相关联
driverscan      驱动程序对象的池扫描器
dumpcerts       转储 RSA 私有和公共 SSL 密钥
dumpfiles       提取内存映射和缓存文件
dumpregistry    将注册表文件转储到磁盘
editbox         显示有关编辑控件的信息。 （列表框实验。）
envars          显示进程环境变量
eventhooks      在 windows 事件挂钩上打印详细信息
                文件对象的文件扫描池扫描程序
gahti           转储 USER 句柄类型信息
gditimers       打印已安装的 GDI 计时器和回调
getservicesids  获取 Registry 中服务的名称并返回计算的 SID
getsids         打印拥有每个进程的 SID
                处理每个进程的打开句柄的打印列表
hashdump        从内存中转储密码哈希 (LM/NTLM)
hibinfo         转储休眠文件信息
hivedump        打印出一个蜂巢
hivelist        打印注册表配置单元列表。
                用于注册表配置单元的 hivescan 池扫描程序
hpaextract      从 HPAK 文件中提取物理内存
hpakinfo        有关 HPAK 文件的信息
iehistory       重建 Internet Explorer 缓存/历史
imagecopy       将物理地址空间复制为原始 DD 映像
imageinfo       识别图像的信息
impscan         扫描对导入函数的调用
joblinks        打印进程作业链接信息
kdbgscan        搜索并转储潜在的 KDBG 值
kpcrscan        搜索并转储潜在的 KPCR 值
ldrmodules      检测 unlin
```

**基本命令格式**
```shell
volatility -f [image] --profile=[profile] [plugin]
```

## 示例
```bash
root@kali:~/vol# volatility -f 1.vmem imageinfo
Volatility Foundation Volatility Framework 2.6
INFO    : volatility.debug    : Determining profile based on KDBG search...
          Suggested Profile(s) : Win7SP1x64, Win7SP0x64, Win2008R2SP0x64, Win2008R2SP1x64_24000, Win2008R2SP1x64_23418, Win2008R2SP1x64, Win7SP1x64_24000, Win7SP1x64_23418
                     AS Layer1 : WindowsAMD64PagedMemory (Kernel AS)
                     AS Layer2 : FileAddressSpace (/root/vol/1.vmem)
                      PAE type : No PAE
                           DTB : 0x187000L
                          KDBG : 0xf800040580a0L
          Number of Processors : 1
     Image Type (Service Pack) : 1
                KPCR for CPU 0 : 0xfffff80004059d00L
             KUSER_SHARED_DATA : 0xfffff78000000000L
           Image date and time : 2020-10-15 06:34:03 UTC+0000
     Image local date and time : 2020-10-15 14:34:03 +0800
```

           

1. 从内存文件中获取用户admin的密码并破解，将该密码作为flag值提交(密码长度为6个字符)；

2. 获取内存文件中系统的IP地址，将IP地址作为flag值提交；

3. 获取内存文件中系统的主机名，将主机名作为flag值提交；  
  
4. 内存文件的系统中存在挖矿进程，将矿池的IP地址作为flag值提交；

5. 内存文件的系统中恶意进程在系统中注册了服务，请将服务名称作为flag值提交。


从内存中转储密码哈希 (LM/NTLM)
```bash
root@kali:~/vol# volatility -f 1.vmem --profile=Win7SP1x64 hashdump
Volatility Foundation Volatility Framework 2.6
Administrator:500:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
Guest:501:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
test:1000:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
admin:1001:aad3b435b51404eeaad3b435b51404ee:acf4a59b219c8eec8aa2ca7b3ba9b7dc:::
```