---
author: coney
comments: true
date: 2013-03-02 15:33:55+00:00
layout: post
slug: bash%e6%8e%a7%e5%88%b6%e8%af%ad%e5%8f%a5%e6%80%bb%e7%bb%93
title: Bash控制语句总结
wordpress_id: 36
categories:
- Linux/Network
---

1. if
``` bash
if [ test ]; then
    xxxx
elif [ test ]; then
    xxxx
else
    xxxx
fi
```

2. for
``` bash
for it in list; do
    xxxx $it
done
```

3. while
``` bash
while [ test ]; do
    xxxx
done
```

4. case
``` bash
case $var in
    "a")
        xxxx;;
    "b")
        xxxx;;
    *)  #default
        xxxx;;
esac
```
