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
BGP报文

1. 应用层报文：BGP公共报文头+BGP packet
	1. 公共报文头
		1. marker 16B
			1. 所有16B都为1，表明BGP的明确开头。
		2. length 2B
		3. Type 1B
			1. packet类型（上面的Type）
				1. open：协商BGP对等体参数，建立对等体关系。在tcp链接之后
				2. update：发送BGP路由更新。在peer建立成功或者有改变
				3. notification：报告错误信息。在运行过程中发现错误时，通告对等体中断peer
				4. keepalive：维护对等体关系
				5. route-refresh：路由策略发生变化时，触发请求对等体重新通告路由
	2. packet内容
		1. open
			1. version（8bit）v4或v6
			2. my as（16bit）
			3. hold time（16bit）默认180s，向下协商
			4. bgp ID（32bit）router id，以IP的形式表示
			5. opt param len
				1. optional param
		2. update——发布或者撤销路由
			1. 