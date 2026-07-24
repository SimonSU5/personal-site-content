---
title: BGP学习笔记02
excerpt: BGP学习笔记02
category: 学习笔记
tags:
  - 路由协议
  - 数通
  - BGP
  - 学习笔记
cover: assets/covers/BGP学习笔记-封面.png
date: 2026-07-21
published: false
---
1. EBGP配置
	1. 使用物理接口
		1. 默认使用物理接口
		2. 默认使用TTL=1
		3. peer 地址 as-number 对端as
	2. 使用环回口
		1. 静态路由添加环回口——跨AS无法使用IGP协议
		2. peer 地址 as-number 对端as
		3. 指定环回口：peer 环回口地址 connect-interface LoopBack x
		4. TTL改为2：peer 环回口地址 ebgp-max-hop 2
2. IBGP配置
	1. 使用物理接口
		1. 默认使用物理接口
		2. 默认使用TTL=256
		3. peer 地址 as-number 对端as
	2. 使用环回口
		1. OSPF协议添加环回口路由
		2. peer 地址 as-number 对端as
		3. 指定环回口：peer 环回口地址 connect-interface LoopBack x
	3. 使用环回口更好，因为网络冗余的情况下，端口关闭还是可以通过冗余路由建立邻居。


---
