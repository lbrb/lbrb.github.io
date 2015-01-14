---
layout: post
title: Ubuntu14 添加系统监控
category: 工具
tags: Linux
keywords: ubuntu，sysmonitor
---

h3. Ubuntu14 利用indicator-sysmonitor在标题栏显示网速，cpu, 内存等信息

具体步骤

*下载包

    wget -c https://launchpad.net/indicator-sysmonitor/trunk/4.0/+download/indicator-sysmonitor_0.4.3_all.deb

*安装依赖

    sudo apt-get install python python-psutil python-appindicator
    
*安装

    sudo dpkg -i indicator-sysmonitor_0.4.3_all.deb

*由于默认的图标在ubuntu中找不到，所以要更改图标。

    sudo vim /usr/bin/indicator-sysmonitor

将 724 行的 sysmonitor 改为刚才记下的 application-community

*执行终端

    indicator-sysmonitor

*开机启动

    mkdir ~/.config/autostart

然后，鼠标右键点击标题栏上 application-community 红心图标，弹出菜单，选择首选项，选中Run on startup。
