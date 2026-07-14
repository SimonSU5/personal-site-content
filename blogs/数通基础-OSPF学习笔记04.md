---
title: OSPF学习笔记04——LSA
excerpt: OSPF学习笔记04——LSA
category: 学习笔记
tags:
  - 路由协议
  - 数通
  - IGP
  - 学习笔记
cover: assets/covers/OSPF学习笔记-封面.png
date: 2026-07-09
published: false
---
## LSA基本概念

- LSA是路由计算的依据
- LSU报文可以携带多种不同类型的LSA
- 各种LSA拥有相同报文头部。
	- IP header | OSPF header | LSU payload
		- LSU payload = LSA header + payload
			- LSA header
				- LS age：LSA生存时间（秒），每1800秒泛洪一次LSA
				- **LS type：LSA类型**
				- **link state id：不同类型的LSA对此字段定义不一样**
				- **advertising router：产生该LSA的路由器router id**
				- LS seq num：LSA每次有新实例产生时，序列号就会增加（就是一个版本号，有更新后，以大seq为准）
			- payload
				- 每种类别都不同
## 常见LSA的类型

| 类型    | 名称                           | 描述                                                                                                                        |
| ----- | ---------------------------- | ------------------------------------------------------------------------------------------------------------------------- |
| 1     | 路由器LSA（Router LSA）           | 每个设备都会产生，描述了设备的链路状态和开销，该LSA只能在接口所属的区域内泛洪                                                                                  |
| 2     | 网络LSA（Network LSA）           | 由DR产生，描述该DR所接入的MA网络中所有与之形成邻接关系的路由器，以及DR自己。该LSA只能在接口所属区域内泛洪                                                                |
| 3     | 网络汇总LSA（Network Summary LSA） | 由ABR产生，描述区域内某个网段的路由，该类LSA主要用于区域间路由的传递                                                                                     |
| 4     | ASBR汇总LSA（ASBR Summary LSA）  | 由ABR产生，描述到ASBR的路由，通告给除ASBR所在区域的其他相关区域。                                                                                    |
| 5     | AS外部LSA（AS External LSA）     | 由ASBR产生，用于描述到达OSPF域外的路由                                                                                                   |
| 7（不学） | 非完全末梢区域LSA（NSSA LSA）         | 由ASBR产生，用于描述到达OSPF域外的路由。NSSA LSA与AS外部LSA功能类似，但是泛洪范围不同。NSSA LSA只能在始发的NSSA内泛洪，并且不能直接进入Area0。NSSA的ABR会将7类LSA转换成5类LSA注入到Area0 |

**查看路由器获得的所有LSA（自己产生的所有LSA）（分类查询）：** dis ospf lsdb router|network|summery|asbr|ase (self-originate)

1. 1类LSA——router LSA（TransNet网络等，以太网广播型网络）
	- 只在区域内部传递
	- 每台OSPF路由器都会产生，他描述了直连口的信息
	- 包含链路类型：P2P，TransNet，StubNet，Vlink
	1. 包内容
		1. LSA header
			1. LS type： router
			2. link state ID：本机router ID
			3. advertising router：产生该LSA的路由器router id（本机router ID）
		2. payload（广播型，只有拓扑信息，没有路由信息）
			1. link ID： DR接口IP地址
			2. link data：OSPF出接口IP地址
			3. link type：TransNet
			4. metric：1
			5. V（virtual link）： 若产生此LSA的路由器为虚链接的端点，则置位
			6. E（external）： 若产生此LSA的路由器为ASBR，则置位
			7. B（border）： 若产生次LSA的路由器为ABR，则置位
			8. links：link数量

| Link Type                                              | Link ID                    | Link Data                |            |
| ------------------------------------------------------ | -------------------------- | ------------------------ | ---------- |
| Point-to-Point (P2P)：描述一个从本路由器到邻居路由器之间的点到点链路，属于拓扑信息    | 邻居路由器的Router ID            | 宣告该Router LSA的路由器接口的IP地址 | 拓扑信息       |
| TransNet：描述一个从本路由器到一个Transit网段（例如MA或者NBMA网段）的连接，属于拓扑信息 | DR的接口IP地址                  | 宣告该Router LSA的路由器接口的IP地址 | 拓扑信息       |
| StubNet：描述一个从本路由器到一个Stub网段（例如Loopback接口）的连接，属于网段信息     | 宣告该Router LSA的路由器接口的网络IP地址 | 该Stub网络的网络掩码             | 路由信息（接口路由） |

2. 2类LSA——network LSA（TransNet网络，DR生成）
	- 由DR产生，描述本网段的链路状态，在所属的区域内传播
	1. LSA header
		1. LS type： network
		2. link state ID：DR接口IP地址
		3. advertising router：产生该LSA的路由器router id（本机router ID）
	2. payload
		1. network mask：MA网络的子网掩码
		2. attached router：连接到该MA网络的所有路由器（包括DR）router id。
3. 