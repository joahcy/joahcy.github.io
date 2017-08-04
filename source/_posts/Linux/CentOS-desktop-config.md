---
title: CentOS桌面切换及配置
date: 2017-06-05
categories: Linux
tags: [Linux,CentOS]

---

## 设置GNOME或者KDE为默认的启动桌面环境
- 方法1: 修改/etc/sysconfig/desktop，根据需要将“DESKTOP”后面的内容改为KDE或GNOME。
- 方法2: 在当前用户目录下建立“.xinitrc”这个文件（注意文件名前有一个点号，代表建立的是一个隐藏文件）,
文件的内容就一行startkde或gnome-session，根据自己的需要选择KDE或GNOME。
## GNOME和KDE的切换
- 1、如果需要切换到GNOME:  
`switchdesk gnome`
- 2、如果需要切换到KDE:  
`switchdesk kde`

## CentOS7设置默认启动为字符界面
- 字符界面：sudo systemctl set-default multi-user.target	
- 图形界面：sudo systemctl set-default graphical.target