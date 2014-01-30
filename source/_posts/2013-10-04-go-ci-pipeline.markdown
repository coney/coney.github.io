---
author: coney
comments: true
date: 2013-10-04 17:44:54+00:00
layout: post
slug: go-ci-pipeline
title: GO CI - 创建Pipeline
wordpress_id: 202
categories:
- Linux/Network
---

# 创建Pipeline


装完了Go肯定要拿来试试效果, 这里使用了过去写过的一个C++小程序来测试整套Build Pipeline系统. 因为这个小程序比较简单, 就分成了两个Stages. 一个是测试, 另一个是编译App.

首先我们给Pipeline起一个名字:

{% img /images/2013/10/Screen-Shot-2013-10-05-at-上午1.08.42.png '' 'Screen Shot 2013-10-05 at 上午1.08.42' %}

点击Next后, 我们需要设置Pipline使用的repo, 这里支持svn, git等多种版本管理工具, 设置好repo地址后继续点击Next.

{% img /images/2013/10/Screen-Shot-2013-10-05-at-上午1.10.32.png '' 'Screen Shot 2013-10-05 at 上午1.10.32' %}

进入第三步后我们要开始配置Pipeline要执行的任务, 这里我们需要填写Stage以及Job的名称. Stage下可以包含多个Jobs. 例如名为Test的Stage下可以包含UnitTest, IntegrationTest等多种测试. 这里只简单运行一下代码中包含的gtest. 因为是c++代码使用Makefile, 这里使用Custom Command来执行我们自己的命令:

{% img /images/2013/10/Screen-Shot-2013-10-05-at-上午1.11.47.png '' 'Screen Shot 2013-10-05 at 上午1.11.47' %}

创建完成后, 可以通过Admin面板下的Pipeline标签查看创建的Pipeline:

{% img /images/2013/10/Screen-Shot-2013-10-05-at-上午1.21.28.png '' 'Screen Shot 2013-10-05 at 上午1.21.28' %}

这里可以修改刚才创建的Stage和Jobs, 也可以增加更多的Stages或者Jobs. 这里额外创建一个Build的Stage, 在Test成功执行后执行.

Pipeline创建完成后, 通过Pipeline面板, 可以查看所有的Pipeline信息, 并进行操作, 点击播放箭头, 开始执行Build:

{% img /images/2013/10/QQ截图20131008230403.jpg '' 'QQ截图20131008230403' %}

Pipeline执行的时间取决于agent的能力以及pipeline中jobs的内容. 当pipeline完成后, 可以点击进度条查看每个Stage执行的结果:

{% img /images/2013/10/build_result.png '' 'build_result' %}



在每个Stage下点击jobs的链接, 可以跳转到对应jobs的详细信息页面:

{% img /images/2013/10/job_details.png '' 'job_details' %}



在Jobs的详细信息页面, 有一个Artifacts标签, 这个下面用于存放pipeline运行过程中生成的一些东西, 可以是日志(例如console.log), 可以是编译后的应用程序, 安装包等(rover_test), 也可以是测试报告等(test_detail.xml), 这里的信息可以在pipeline admin中jobs对应的artifacts配置中修改:{% img /images/2013/10/job_config_artifacts.png '' 'job_config_artifacts' %}

这里的Artifacts类型分为两种, Build的是一般产出的程序, 压缩包之类的东西, 可以供下载使用, 而Test Artifact则是由Go进行解析的报告, 例如这个示例中通过gtest的--gtest_output=xml选项生成了XML格式的日志, 被Go解析后可以直接在Job的运行结果页面看到执行的测试结果:

{% img /images/2013/10/test_result.png '' 'test_result' %}



不过这里发现的一个问题就是同一个pipeline中前面jobs产生的artifact不能直接被后面的job使用, 不知道是没找到方法还是真的不支持. 总体来说用起来不是特别复杂, 但是觉得跟bamboo比起来还是有差距,
