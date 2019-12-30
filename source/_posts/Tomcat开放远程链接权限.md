---
title: Tomcat开放远程链接权限
date: 2019-05-12 12:53:35
categories:
- Code
tags:
- 运维
- Tomcat
banner_img: 
index_img: 
---

# Tomcat8及以上版本开放远程连接权限

在使用jenkins进行自动化部署的过程中，发现无法连接到目标的tomcat。经过查阅后整理一下文档：

## 测试权限是否已开放

尝试能否进入容器下的manger地址


    http://yourIP:8080/manger



## 添加一些权限以及一个连接使用的用户

对/usr/local/apache-tomcat-8.5.33/conf文件夹下的tomcat-users.xml进行编辑，添加一下内容：


    <role rolename="manager"/>
    <role rolename="manager"/>
    <role rolename="manager-gui"/>
    <role rolename="admin"/>
    <role rolename="admin-gui"/>
    <role rolename="manager-script"/>  
    <user username="tomcat" password="tomcat" roles="admin-gui,admin,manager-gui,manager,manager-script"/>


## 配置允许连入的IP

在`/usr/local/apache-tomcat-8.5.33/webapps/manager/META-INF`目录下编辑content.xml文件配置允许连接的ip地址(`将allow值改成"^.*$"表示所有ip都可以访问`)：

修改以前：

    <Valve className="org.apache.catalina.valves.RemoteAddrValve" allow="127\.\d+\.\d+\.\d+|::1|0:0:0:0:0:0:0:1" />


修改以后：


    <Valve className="org.apache.catalina.valves.RemoteAddrValve"allow="^.*$" />


