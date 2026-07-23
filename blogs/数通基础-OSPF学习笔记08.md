---
title: OSPF学习笔记08——路由汇总和其他
excerpt: OSPF学习笔记08——路由汇总和其他
category: 学习笔记
tags:
  - 路由协议
  - 数通
  - IGP
  - 学习笔记
cover: assets/covers/OSPF学习笔记-封面.png
date: 2026-07-21
published: false
---
## 路由汇总
在ABR上的area，执行abr-summery，将多个网段合并成一个网段进行summery
在ASBR上，执行asbr-summary
在NSSA ASBR上，执行asbr-summery
在NSSA的translator上，执行asbr-summery

## silent-interface
在路由器直连末端设备上设置silent-interface。可以让路由器不往末端设备上发送ospf报文，节省末端设备的带宽和负担。

### OSPF报文认证
1. 区域认证方式:一个osPF区域中所有的路由器在该区域下的认证模式和口令必须一致。
2. 接口认证方式:相邻路由器直连接口下的认证模式和口令必须一致。
	- authentication-mode [md5 1 /simple] cipher xxx密码