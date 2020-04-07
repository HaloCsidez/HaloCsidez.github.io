---
title: Tomcat进程守护脚本
date: 2020-02-24 13:46:30
tags: 
- TOMCAT 
- SHELL
---
# Tomcat进程守护脚本

迫于公司测试服务器上的几个子站经常时不时的挂掉，又查不明原因。
谷歌了以后说要把jdbc移出来放到tomcat的lib目录下，让tomcat进行管理。可是怎么实验都没有，恰巧正式服务器上的进程又是好的，于是，有了一下的进程守护脚本。

## 使用环境
- 操作系统:CentOS7.2
- JDK版本:Java HotSpot(TM) 64-Bit Server VM (build 24.80-b11, mixed mode)
- Tomcat版本:apache-tomcat-7.0.70

## 监控脚本

	#!/bin/bash
	#
	# mail：zhouhaibin@utry.cn
	# 定时检查tomcat状态
	# 当在进程中查询不到该tomcat则进行重启
	# 
	# 需要添加至定时任务crontab中
	# 1. crontab -e
	# 2. */3 * * * * "/home/test/deamon2tomcat.sh" > /dev/null 2>&1
	# 注：每3分钟检查一次(可修改)，加“> /dev/null 2>&1 ”是为了不让它发邮件。

	export PATH=$PATH:/sbin:/bin:/usr/sbin:/usr/bin;
	tomcat_base_dir="/home/tomservices";
	log_file="/home/tomservices/tomcat_protect_log.log";
	d=$(date +%F" "%T);

	# 初始化日志文件
	touch ${log_file}

	do_restart_tomcat() {
		# 不管是否再运行中先关闭服务
		${tomcat_base_dir}/$1/bin/shutdown.sh
		
		# 在进程中检查当前当tomcat是否正在运行并杀死该进程（为了防止shotdown.sh失败）
		if ps -ef | grep $1 | grep -v grep
		then
			kill -9 $(ps -ef | grep $1 | grep -v grep | awk '{print $2}')
		fi
		
		# 重启tomcat
		${tomcat_base_dir}/$1/bin/startup.sh
		# 输出进程号
		echo "$d success restart $1, the new pid is " >> ${log_file}
		ps -ef | grep $1 | grep -v grep | awk '{print $2}' >> ${log_file}
		echo >> ${log_file}
	}

	echo "$d start check the three server:" >> ${log_file}
	# 循环检查一下三个子站是否正在运行并输出相应日志
	for n in "tomcat_devback" "tomcat_hsipccweb" "tomcat_ccweb"
	do
	echo "$d checking the $n..." >> ${log_file}
		if ps -ef | grep $n | grep -v grep
		then
			echo "$d finished the check of the $n" >> ${log_file}
		else
			echo "$d can not found $n, try to restart it ......" >> ${log_file}
			do_restart_tomcat $n
		fi
	done

## 加入定时任务crontab
### 编辑任务
```
crontab -e
```
进入编辑crontab的vi界面，按i键进入插入模式，将如下代码复制到输入界面
```
*/20 * * * * "/home/test/deamon2tomcat.sh" > /dev/null 2>&1
```
`注：每20分钟检查一次(可修改)，加“> /dev/null 2>&1 ”是为了不让它发邮件。`
### 保存退出
### 查看任务
最后查看任务，确认是否编辑成功。
```
crontab -l
```
