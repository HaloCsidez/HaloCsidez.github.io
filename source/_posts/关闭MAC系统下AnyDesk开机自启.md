---
title: 关闭MAC系统下AnyDesk开机自启
date: 2019-06-29 14:54:07
tags:
- 运维
- MAC系统
---
## 视图内关闭开机自动启动
`系统偏好设置->用户与组->登陆项`中进行检查，检查应用是否有勾选

## 通过命令行进行关闭开机自启
以AnyDesk为例：

### 检查plist文件
分别在以下6个目录中检查是否有与anydesk相关的plist文件
1. /Library/Preferences/  – （当前用户设置的进程）
2. /Library/LaunchAgents/  – （当前用户的守护进程）
3. /Library/LaunchAgents/  – （管理员设置的用户进程）
4. /Library/LaunchDaemons/  – （管理员提供的系统守护进程）
5. /System/Library/LaunchAgents/ – （Mac操作系统提供的用户进程）
6. /System/Library/LaunchDaemons/   – （Mac操作系统提供的系统守护进程）

### 修改plist文件
按要求检查出以下三个与anydesk相关的plist文件，分别修改以下三个文件中的内容并保存。
1. /Library/LaunchAgents/com.philandro.anydesk.Tray.plist

        <key>KeepAlive</key>
        <false/>
        ...
        <key>RunAtLoad</key>
        <false>


2. /Library/LaunchDaemons/com.philandro.anydesk.Helper.plist

        <key>com.philandro.anydesk.Helper</key> #关闭应用相关的工具
        <false>

3. /Library/LaunchDaemons/com.philandro.anydesk.service.plist

        <key>RunAtLoad</key>
        <false/>
        <key>KeepAlive</key>
        <false>

### 关闭服务
打开命令行终端（Terminal）执行以下命令：

    # 查看anydesk服务名
    launchctl list | grep anydesk
    # 停止服务
    launchctl stop com.philandro.anydesk
    # 移除服务
    launchctl unload com.philandro.anydesk