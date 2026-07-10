---
title: OSPF学习笔记04——LSA
excerpt: OSPF学习笔记04——LSA
category: 数通
tags:
  - 路由协议
  - 数通
  - IGP
cover: assets/covers/image.jpg
date: 2026-07-09
published: false
---
## LSA基本概念

- LSA是路由计算的依据
- LSU报文可以携带多种不同类型的LSA
- 各种LSA拥有相同报文头部。
	- IP header | OSPF header | LSU payload
		- LSU payload = LSA header + payload
			- LS age：LSA生存时间（秒），每1800秒泛洪一次LSA
			- **LS type：LSA类型**
			- **link state id：不同类型的LSA对此字段定义不一样**
			- **advertising router：产生该LSA的路由器router id**
			- LS seq num：LSA每次有新实例产生时，序列号就会增加（就是一个版本号，有更新后，以大seq为准）
## 常见LSA的类型

| 类型    | 名称                           | 描述                                                                                                                        |
| ----- | ---------------------------- | ------------------------------------------------------------------------------------------------------------------------- |
| 1     | 路由器LSA（Router LSA）           | 每个设备都会产生，描述了设备的链路状态和开销，该LSA只能在接口所属的区域内泛洪                                                                                  |
| 2     | 网络LSA（Network LSA）           | 由DR产生，描述该DR所接入的MA网络中所有与之形成邻接关系的路由器，以及DR自己。该LSA只能在接口所属区域内泛洪                                                                |
| 3     | 网络汇总LSA（Network Summary LSA） | 由ABR产生，描述区域内某个网段的路由，该类LSA主要用于区域间路由的传递                                                                                     |
| 4     | ASBR汇总LSA（ASBR Summary LSA）  | 由ABR产生，描述到ASBR的路由，通告给除ASBR所在区域的其他相关区域。                                                                                    |
| 5     | AS外部LSA（AS External LSA）     | 由ASBR产生，用于描述到达OSPF域外的路由                                                                                                   |
| 7（不学） | 非完全末梢区域LSA（NSSA LSA）         | 由ASBR产生，用于描述到达OSPF域外的路由。NSSA LSA与AS外部LSA功能类似，但是泛洪范围不同。NSSA LSA只能在始发的NSSA内泛洪，并且不能直接进入Area0。NSSA的ABR会将7类LSA转换成5类LSA注入到Area0 |

