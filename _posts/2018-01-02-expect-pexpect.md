---
layout:     post
title:      "自动化处理人机交互"
subtitle:   "expect和pexpect的简单使用"
date:       2018-01-02 00:36:00
author:     "lyremelody"
header-img: "img/post-bg-2015.jpg"
catalog: true
tags:
    - 运维
---


我们在Linux系统下，习惯用命令行来做一些操作，然后通过脚本将一系列命令组合起来做一些自动化的事情，为了...懒…

但是有时候会遇到问题，比如你在脚本中需要yum安装某些工具，比如
> yum install expect 

它会卡住，类似

    ………
    Total download size: 2.2 M
    Installed size: 4.9 M
    Is this ok [y/N]:


这种其实还好，有些命令行工具提供了 -y 参数可以做，比如
> yum install -y expect

但是还有一些就没那么幸运了，比如
> ssh root@lyremelody.org

结果可能是这样：

    [lyremelody@lyremelody-mba ~]$ ssh root@lyremelody.org
    The authenticity of host 'lyremelody.org (121.199.38.168)' can't be established.
    RSA key fingerprint is SHA256:97yhGT78mEGnTF4ifsZpe9RG0giw2wDqwoMcrumtekY.
    Are you sure you want to continue connecting (yes/no)? 

这种需要人机交互的命令行，还有很多，比如fsck、ftp、telnet等。对于自动化造成了很大的“伤害”（主要是对需要做自动化的人）。

## Expect
---
好吧，其实是有工具能做的。Don Libes实现了Unix下进行自动化控制和测试的软件工具expect，作为Tcl脚本语言的扩展，能够通过Unix伪终端包装子进程，捕获并处理子进程的输出，并可以对其进行响应，以实现自动化交互。

当然Linux下也是可以用的。

先要安装expect，如：
> yum install -y expect

它会自动安装依赖，比如tcl。

现在对于ssh的自动化登录，可以这么做了：

    #!/usr/bin/expect
    spwan ssh root@lyremelody.org
    expect "(yes/no)? "
    send "yes\r"
    expect "password: "
    send "my_password"
    expect "*#”
    interact

执行这个，你就能通过ssh自动登录到站点`lyremelody.org`，不需要在过程中再输入"yes"和密码了。

对于简单的场景，这样就够用了，复杂一些的，通过Shell脚本可能就不好管理了。自然我们想到了Python。

## Pexpect
---
Pexpect基于纯Python实现，类似Don Libes的Expect，能够启动子进程，捕获并匹配子进程输出，从而进行响应，以实现与子进程的交互。类似实现人机交互的问答式的命令处理。

Pexpect可以实现自动交互的应用包括ssh、ftp、telnet等。

Pexpect基于纯Python实现，不需要依赖expect、tcl或者其他C库。跨平台特性好，依赖Python。

当前基于pexpect 4.3.1版本讨论。

### Pexpect安装
基于Python 3.3及以上，或者Python 2.7
> pip install pexpect==4.3.1

### Pexpect使用
Pexpect跟expect类似，主要可以做三件事情：
1. 创建子进程、执行命令 （通过pexpect.spwan）
2. 匹配子进程输出 (expect / expect_exact / expect_list)
3. 向子进程发送输入 (send / sendline / sendcontrol / sendeof / sendintr)

通过循环匹配进程输出和向子进程发送输入来进行自动化人机交互。

通过 expect_list() 匹配预期操作成功和失败的字符串，可以实现错误处理。
如
 
    res = expect_list ([‘ftp>', 'Permission denied, please try again.'])
    if res == 0:
        # 登录成功，接着操作
    elif res == 1
        # 登录失败，重新输入密码

expect() 执行过程中有两类异常
* 匹配终止 (pexpect.EOF)
* 超时 (pexpect.TIMEOUT)

通过 expect() 和 pexpect.TIMEOUT 配合起来用，可以实现匹配失败的处理。

下面我们看一个基于pexpect实现访问ftp自动化的例子。

    # 主要实现的功能：
    # 连接到 ftp.openbsd.org ftp站点  (例子中的站点实际上不能访问，仅做例子)
    # 通过匿名登录   
    # 下载 pub/OpenBSD/README 文件到本地 /tmp 目录
    import pexpect 

    # pexpect 创建一个子进程 ftp 连接 ftp.openbsd.org，并监控这个子进程的输出，控制子进程的输入
    child = pexpect.spawn('ftp ftp.openbsd.org')
    
    # pexpect 匹配上面连接信息返回的内容是否通过正则匹配 "`Name .*: `"，如果没有，则会抛异常，比如超时 pexpect.TIMEOUT，这里不处理
    child.expect('`Name .*: `')
    
    # 上面匹配成功后，发送“anonymous”行到子进程 (ftp进程)，ftp进程发送到远程进行用户名验证
    child.sendline('anonymous')
    
    # pexpect 等待ftp进程输出，匹配是否包含"Password:"
    child.expect('Password:')
    
    # 上面匹配成功后，发送"noah@example.com”到ftp进程，ftp进程发送到远程进行登录
    child.sendline('noah@example.com')
    
    try:
    # pexpect 等待ftp进程输出，匹配是否包含字符串"ftp>”或者"Permission denied"
    res = child.expect_list(['ftp> ', 'Permission denied'])
    if res == 0:
        # 匹配"ftp>"，表示登录成功
        pass
    elif res == 1:	
        # 匹配 "Permission denied"，密码错误，可做相应处理
        return
    except pexpect.TIMEOUT:
        # 未匹配到，超时异常
        raise
    
    # pexpect 发送字符串 lcd /tmp和换行符，将本地目录切换到 /tmp
    child.send('lcd /tmp\n')
    
    # pexpect 等待ftp进程输出，匹配是否包含字符串"ftp>"，匹配则表示之前的命令执行完成
    child.expect_exact('ftp> ')
    
    # pexpect 发送 cd pub/OpenBSD 到ftp进程，将远程目录切换到 pub/OpenBSD
    child.sendline('cd pub/OpenBSD')
    
    # pexpect 等待ftp进程输出，匹配是否包含"ftp>"，匹配则表示之前的命令执行完成
    child.expect('ftp> ')
    
    # pexpect 发送 get README 到ftp进程，将远程的README文件下载到本地
    child.sendline('get README')
    
    # pexpect 等待ftp进程输出，匹配是否包含”ftp>"，匹配则表示之前的命令执行完成
    child.expect('ftp> ')
    
    # pexpect 发送 bye 到ftp进程，退出ftp连接
    child.sendline('bye')

这样就可以通过Python作为胶合层来管理各种需要人机交互的东西了。

expect和pexpect适用于自动化过程中需要与系统命令行交互的场景。

expect和pexpect你值得拥有～ 


