---
layout: post
title: "移除HTTP Proxy的NTLM认证"
date: 2015-08-07 00:04:58 +0800
comments: true
categories:
---

Node.js需要下载安装`Karma`, `Jasmine`等组件来运行UT, 但是华为访问网络需要通过NTLM认证的代理, npm不支持NTLM, 需要通过CNTLM工具将代理转化为不需要认证的HTTP代理.

从[CNTLM](http://cntlm.sourceforge.net/)官方网站下载最新版本的CNTLM程序:

http://sourceforge.net/projects/cntlm/files/

<!--more-->

安装CNTLM后, 我们需要对CNTLM进行一些配置.
首先进入CNTLM安装目录, 打开`cntlm.ini`, 修改代理服务器地址为目前使用的NTLM认证的代理服务器地址:

```
Proxy     your.proxy.com:8080
Listen    3128
```

需要注意此处是**修改**`cntlm.ini`中的对应项而不是新增, 需要移除掉原来的`Listen`和`Proxy`配置.

之后运行`cmd`进入命令行环境, 切换到CNTLM安装目录, 执行如下命令:

```
C:\Program Files\Cntlm> cntlm -M http://www.baidu.com/ -d your.domain.com -u username
Password:
Config profile  1/4... Credentials rejected
Config profile  2/4... OK (HTTP code: 200)
----------------------------[ Profile  1 ]------
Auth            NTLM
PassNT          6549XXXXXXXXXXXXXXXXXXXXXXXXC0B0
PassLM          1B1AXXXXXXXXXXXXXXXXXXXXXXXXBFE3
------------------------------------------------
```

将`-u`后的参数换成你的域账号, 并且在提示输入Password的地方输入域账号密码.
cntlm会尝试使用NTLM认证, 并且输出了认证使用的Key Hash.
编辑CNTLM安装目录的`cntlm.ini`, 根据刚才返回的Key Hash, 修改以下几行:

```
Username  username
Domain    your.domain.com

Auth      NTLM
PassNT    6549XXXXXXXXXXXXXXXXXXXXXXXXC0B0
PassLM    1B1AXXXXXXXXXXXXXXXXXXXXXXXXBFE3
```

最后启动CNTLM服务:

```
net start cntlm
```

在浏览器中配置代理`127.0.0.1:3128`, 尝试访问外网, 验证代理是否工作正常.
