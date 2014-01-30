---
author: coney
comments: true
date: 2013-10-02 13:46:56+00:00
layout: post
slug: go-ci-setup
title: GO CI - 安装配置
wordpress_id: 177
categories:
- Linux/Network
---

什么是Go呢? Go是骚窝出品的一款持续交付(CD)产品, 类似于Hudson, Bamboo等CI服务. 作为一家专业的敏捷和持续交付咨询公司,  Go从定位上就已经拉开了与其他同类产品的距离, 当别人还在搞CI的时候, 骚窝已经在玩CD了. 下面简单贴下安装和配置流程, 以及试用体验.


# 安装Go Server
<!-- more -->

首先我们要到Go的产品页面去下载安装包([请猛击此处](http://www.thoughtworks.com/products/go-continuous-delivery)). 目前Go提供一个免费的社区版, 限制是最多10个用户, 3个远程Agent. 这个对个人使用感觉已经足够了. 来到Go的下载页面, 能看到Go提供了多种安装包, 针对不同的系统, 因为我的VPS是Ubuntu的, 所以点击Linux(Debian/Ubuntu)下载了deb包, 如果使用的是CentOS等使用RPM的发行版, 可以点击Linux(Redhat/Suse)这一项下载rpm包. 当然如果您使用非linux系统可以选择其他类型的安装包:

{% img /images/2013/10/Screen-Shot-2013-10-04-at-下午11.43.37.png '' 'Screen Shot 2013-10-04 at 下午11.43.37' %}来进行操作管理的服务, 而Go Agent则是执行Go Server下发的工作的苦力. 要实现完整的CI系统, 我们需要同时安装和配置Server以及Agent. 一个Go Server可以拥有多个Agent, 而且这些Agent可以跟Server安装在一起, 也可以安装在其他服务器上.

下载好安装包后, 我们首先来安装Go Server. 以我使用的Ubuntu Server为例, 首先通过Dpkg来进行安装:

    
    ╭─coney@UServer-Laptop  ~
    ╰─% sudo dpkg -i go-server-13.2.2-17585.deb
    Selecting previously unselected package go-server.
    (Reading database ... 102421 files and directories currently installed.)
    Unpacking go-server (from go-server-13.2.2-17585.deb) ...
    No packages found matching cruise-server.
    Adding system user `go' (UID 108) ...
    Adding new group `go' (GID 118) ...
    Adding new user `go' (UID 108) with group `go' ...
    Creating home directory `/var/go' ...
    Setting up go-server (13.2.2-17585) ...
    Found Java /usr/lib/jvm/java-7-openjdk-amd64/jre in PATH, using it.
    chown: cannot access `/var/lib/go-server/**': No such file or directory
    Installation of Go Server completed.
    using default settings from /etc/default/go-server
    nohup: redirecting stderr to stdout
    Started Go Server on http://UServer-Laptop:8153/go
    Processing triggers for ureadahead ...
    ureadahead will be reprofiled on next reboot


虽然看起来像是有些错误, 但实际还是安装成功了, 根据提示, 我们访问一下服务器的8153端口, It works!

{% img /images/2013/10/Screen-Shot-2013-10-05-at-上午12.24.32.png '' 'Screen Shot 2013-10-05 at 上午12.24.32' %}


#  配置用户认证


初次进入Go, 并没有见到登录验证之类的页面, 而是直接跳转到Admin下让我们构建pipeline. 如果只是个人使用的话, 这样还是挺方便的, 但是如果发布到公网或者有多个用户时, 这样就太不安全啦,  所以首要任务是加入登录验证, 在[Admin][Server Configuration]标签下, 能够找到User Management相关的配置选项:

{% img /images/2013/10/Screen-Shot-2013-10-05-at-上午12.31.39.png '' 'Screen Shot 2013-10-05 at 上午12.31.39' %}

参考配置文档的说明, 支持LDAP和密码文件认证. 因为没有AD服务器, 所以这里使用的第二种方式. Go要求密码要通过sha1哈希后在通过base64编码一次. 手动操作比较繁琐, 这里用apache提供的htpasswd命令生成和修改密码:

    
    ╭─coney@UServer-Laptop  /var/lib/go-server
    ╰─% sudo htpasswd -s -c passwd coney
    New password:
    Re-type new password:
    Adding password for user coney


参数-c表示创建新的密码文件, passwd是文件名, coney是要添加的第一个用户, 命令执行后会提示输入和确认密码. 输入无误便会创建用户. 之后将新创建的文件路径填入"Password File Path"一栏中提交. 提交后便会跳转到登陆页面, 此时通过刚才创建的用户名密码登陆, 重新进入Go主页.

{% img /images/2013/10/Screen-Shot-2013-10-05-at-上午12.49.56.png '' 'Screen Shot 2013-10-05 at 上午12.49.56' %}


#  安装Go Agent


在下载Go Server时, 大家应该看到了Go Agent的下载链接. 如果没有Agent的话, 在Go上配置的Pipeline将都不能执行. 现在我们用类似的方法安装Agent, 同样我这里下载的是Go Agent的deb包:

    
    ─coney@UServer-Laptop  ~
    ╰─% sudo dpkg -i go-agent-13.2.2-17585.deb
    Selecting previously unselected package go-agent.
    (Reading database ... 102435 files and directories currently installed.)
    Unpacking go-agent (from go-agent-13.2.2-17585.deb) ...
    No packages found matching cruise-agent.
    'Go Agent' installation will now use group 'go'
    'Go Agent' installation will now use user 'go'
    Setting up go-agent (13.2.2-17585) ...
    Found Java /usr/lib/jvm/java-7-openjdk-amd64/jre in PATH, using it.
    Installation of Go Agent completed.
    Now please edit /etc/default/go-agent and set GO_SERVER to the IP address of your Go Server.
    Once that is done start the Go Agent with '/etc/init.d/go-agent start'
    Processing triggers for ureadahead ...


Agent的安装速度相当快, 根据安装的提示, 我们要编辑/etc/default/go-agent这个配置文件, 并设置GO_SERVER的地址. 打开go-agent配置, 发现GO_SERVER当前的值为127.0.0.1, 因为我在同一台服务器上安装的Go Server和Go Agent, 所以又可以偷个懒直接使用啦. 如果在不同的主机上安装Agent, 需要将GO_SERVER指向真实的Go Server地址, 并且保证Server和Agent间的网络互通.

需要注意的是默认安装完后Go Agent是没有启动的, 我们需要手动启动它:

    
    coney@UServer-Laptop  ~
    ╰─% /etc/init.d/go-agent status
    Go Agent is stopped.
    ╭─coney@UServer-Laptop  ~
    ╰─% sudo /etc/init.d/go-agent start
    [Sat Oct  5 00:59:13 CST 2013] using default settings from /etc/default/go-agent
    nohup: redirecting stderr to stdout
    Started Go Agent.


启动成功后, 我们在Go Server Agent面板下能够看到现在可用的Agents列表:

{% img /images/2013/10/Screen-Shot-2013-10-05-at-上午1.03.41.png '' 'Screen Shot 2013-10-05 at 上午1.03.41' %} 到此为止, 一个基本可用的Go就算搭建完成.
