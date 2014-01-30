---
author: coney
comments: true
date: 2013-02-19 12:19:52+00:00
layout: post
slug: '%e9%80%9a%e8%bf%87ldd%e6%9f%a5%e6%89%be%e7%ac%a6%e5%8f%b7%e5%bc%95%e7%94%a8'
title: 通过ldd查找符号引用
wordpress_id: 11
categories:
- Linux/Network
---
{% img /images/2013/02/无标题.png '' 'ldd man' %}
根据ldd manual, 通过-d, -r分别可以查看缺失的符号, 对于elf格式的so一样有效, 例如:
    [~/qmf/application/server/qmf_cmdsvr]$ ldd -r src/qmfcmd_spp.so
    linux-gate.so.1 => (0xbfffe000)
    libpthread.so.0 => /lib/libpthread.so.0 (0xb74cf000)
    libstdc++.so.6 => /usr/lib/libstdc++.so.6 (0xb73eb000)
    libm.so.6 => /lib/libm.so.6 (0xb73c3000)
    libgcc_s.so.1 => /lib/libgcc_s.so.1 (0xb73b7000)
    libc.so.6 => /lib/libc.so.6 (0xb7276000)
    /lib/ld-linux.so.2 (0x80000000)
    undefined symbol: _ZN5tbase4tlog5CTLog5log_iEiiPKcz (src/qmfcmd_spp.so)





