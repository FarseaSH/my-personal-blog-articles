---
date: 2017-12-25 16:32:02
title: 在Windows下清除ssh-key私钥访问密码
tags:
 - Linux
 - Windows
category: 经验
showupdate: true
updated: 2018-06-16
copyright: true
---

之前在配置Github时候，给本地生成的ssh-key加上了访问密码，因为所有ssh连接均采用了同一个key，给之后访问带来了很多的不便，查询相关资料找到取消该访问密码的方式。
<!--more-->

首先，需要打开git-bash，（**注意**这里是不能使用Windows系统自带的终端或者powershell）输入命令

```
ssh-keygen -p

Enter file in which the key is(/c/Users/YourUserName/.ssh/id_rsa):
```

命令会自动定位到默认生成的私钥，输入原始密码，在提示输入新密码时候，直接2次回车确认即可。