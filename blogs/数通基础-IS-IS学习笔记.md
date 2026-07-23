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
	2. 1：区域号
	3. 128：内部接口可达性
	4. 129：支持的协议
	5. 130：外部接口可达性
	6. 132：IP接口地址
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
8. level 1路由器
	1. 区域内部路由器
	2. 只可以和**同一区域**的level1路由器或者**同一区域**的level1-2路由器建立邻接关系
	3. 发送的level1报文，建立level1 lsdb
9. ISIS支持的网络类型
	1. 广播
	2. 点到点——串口
10. cost开销
	1. 不和带宽绑定，缺省为10
	2. 设定方式
		1. 接口开销
		2. 全局开销
		3. 自动计算开销——根据带宽自动计算开销（）
			1. 2.5G以上为10
			2. (622M, 2.5G]为20
			3. (155M,622M]为30
			4. (100M,155M]为40
			5.  (10M,100M]为50
			6. (0, 10M]为60

---
1. 邻居关系建立要求
	1. 只有同一层次的相邻路由器才有可能成为邻接
	2. 对于level1路由器来说，area ID一定要一致
	3. 链路两端接口的网络类型必须一致（p2p，广播）
	4. 链路接口地址必须在同一网段
2. IIH报文 IS-IS hello报文
	1. holding time：默认30秒，30s没有收到hello报文，则断开链接
	2. priority：选举DIS。取值范围0-127，数值越大越优先。优先级一样，选取mac大的。只有LAN 的IIH才会携带
	3. circuit：
3. 邻接接关系建立过程
	1. 所有设备都会建立邻接关系
	2. R1发送IIH（sys id，area，neighbor）-R1，R2initial
	3. R2发送IIH（sys id，area，neighbor）-R1 up
	4. R1发送IIH（sys id，area，neighbor）-R2 up
	5. 两次hello报文后，进行DIS选举
4. DIS选举
	1. 选举出实节点
	2. 实节点生成伪结点
	3. 伪结点生成伪结点LSP——对比ospf2类LSA
	4. 实节点生成LSP——对比ospf1类LSA
	5. 伪结点LSP：使用伪结点DIS的sys ID以及circuit ID（非0值）标识。
	6. 所有路由器都与伪结点建立邻接关系
	7. 所有路由器都必须参选（对比OSPF DR优先级为0不参与），优先级大的为DIS，优先级一样的情况下MAC大的为DIS。
	8. DIS发送IIH的频率是普通的3倍。这样能够更快发现DIS的故障。
	9. DIS可以被抢占
	10. 同一网段同一级别都两两建立邻接关系。
	11. 01开头的（第8bit为1）为组播地址。目的地址为00:14结尾的，为level 1 的组播地址，00:15
