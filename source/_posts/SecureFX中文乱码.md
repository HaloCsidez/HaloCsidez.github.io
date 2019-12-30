---
title: SecureFX中文乱码
date: 2019-06-29 22:18:20
tags:
- MAC系统
- 软件
---
## 解决中文乱码问题

1. `Options->Global Options->General->Default Session`，找到配置目录；

2. 进入该配置目录的Sessions文件夹，修改对应的.ini文件，找到包含：

        D:"Filenames Always Use UTF8"=00000000

   的这一行，修改配置文件内容为：

        D:"Filenames Always Use UTF8"=00000001 

   即可；

3. 重启SecureFX，再次连接，中文就显示正常了。

