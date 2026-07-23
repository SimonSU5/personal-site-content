---
title: IS-IS学习笔记
excerpt: IS-IS学习笔记
category: 学习笔记
tags:
  - 路由协议
  - 数通
  - IGP
  - 学习笔记
cover: assets/covers/IS-IS学习笔记-封面.png
date: 2026-07-21
published: false
---
1. 数据链路层
2. PDU通用报文头（相同）+PDU专用报文头+TLV（type，length，value）+payload
3. 类型：
	1. IIH：isis hello
		1. level 1 LAN IIH
		2. level 2 LAN IIH
		3. P2P IIH
	2. LSP（link state PDU）：链路状态报文 类似于OSPF LSU
		1. level 1 LSP
		2. level 2 LSP
	3. CSNP（全序列号报文）：类似于OSPF DD
		1. level 1 CSNP
		2. level 2 CSNP
	4. PSNP（部分序列号报文）：类似于LSR，LSA
		1. level 1 PSNP
		2. level 2 CSNP
4. TLV
	1. type 2 LSP：中间系统邻接
	2. 128：内部接口可达性
	3. 130：外部接口可达性
	4. 132：IP接口地址
5. NET地址
	1. 特殊的NSAP地址（ISO的IP）
	2. AREA ID（区域）+System ID（Router id）+SEL（=00）
	3. 长度和NSAP相同，最长20B，最短8B
	4. 区域ID长度1-13B，system ID长度6B，SEL长度1B
6. NET配置
	1. 每个设备至少有一个NET。可以有多个NET，但是system ID必须相同。一般根据router ID配置system ID
7. ISIS拓扑
	1. 骨干区域路由器：level 2
	2. 区域边缘路由器（也属于骨干区域）：level 1-2 （缺省情况）
	3. 非骨干区域：level 1