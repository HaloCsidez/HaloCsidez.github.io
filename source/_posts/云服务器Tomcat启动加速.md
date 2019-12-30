---
title: 云服务器加快Tomcat启动速度
date: 2019-05-28 16:31:42
tags:
- Tomcat
- 运维
- 云服务器
---
解决阿里云服务器上的Tomcat启动巨慢的问题
<!-- more -->
### 配置
- 阿里云学生套餐
- Tomcat8.0
- CentOS7

### 问题现象
- 启动过程会停留在：
`Deplopying web application directory...`
需要等待很久才能完成

### 问题分析
- 借助strace工具（跟踪系统调用），利用代码`strace -f -o strace.out ./catalina.sh run`
- 分析过程请参照 [原文](https://www.jianshu.com/p/576d356dc163)

### 主要原因
- Tocmat在启动过程中的Session ID是通过SHA1算法计算得到的，计算Session ID的时候必须有一个密钥。
- 为了提高安全性Tomcat在启动的时候回通过随机生成一个密钥。
- 而随机数的生成是通过/dev/random（Linux下的随机函数生成器）来实现的。
- 然而，/dev/random会根据噪音产生随机数，如果噪音不够它就会阻塞。Linux是通过I/O，键盘终端、内存使用量、CPU利用率等方式来收集噪音的，如果噪音不够生成随机数的时候就会被阻塞。（源自wiki百科）

### 解决方案
- `用伪随机函数生成器(/dev/urandom)来替代随机函数生成器(/dev/random)`，可以有一下两种办法：
    1. 通过修改JRE中的java.security（/usr/java/jdk1.8.0_121/jre/lib/security/java.security）文件securerandom.source=file:/dev/urandom
    2. java程序启动参数中添加：-Djava.security.egd=file:/dev/urandom，使用/dev/urandom生成随机数。

- `增大/dev/random的熵池`
    - 通过cat /proc/sys/kernel/random/entropy_avail我们可以查看现在的熵池大小
    - 通过at /proc/cpuinfo | grep rdrand查看CPU是否支持DRNG特性
    - 1. 支持DRNG特性：
        
                yum install rngd-tools（yum install rng-tools）
                /*安装rngd服务（熵服务）*/
                systemctl start rngd

    - 2. 不支持DRNG特性或者使用虚拟机，可以使用/dev/unrandom来模拟：

                cp /usr/lib/systemd/system/rngd.service /etc/systemd/system
                编辑/etc/systemd/system/rngd.service
                修改ExecStart参数为/sbin/rngd -f -r /dev/urandom
                systemctl daemon-reload
                重新载入服务
                systemctl restart rngd
                重启服务

    - 再次查看熵池大小，可以利用watch -n 1 cat /proc/sys/kernel/random/entropy_avail测试随机数生成速度
