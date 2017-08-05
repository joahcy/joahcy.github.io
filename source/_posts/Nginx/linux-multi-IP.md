---
title: linux配置多IP
date: 2017-06-01
comments: false
categories: Linux
tags: Linux

---

- 1.复制第一块网卡的配置文件,例:  
`cp  ifcfg-eth0  ifcfg-eth0:1`
- 2.修改ifcfg-eth0:1的DEVICE 和 IPADDR 
- 3.重启虚拟机