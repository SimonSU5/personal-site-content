---
title: BGP学习笔记01
excerpt: BGP学习笔记01
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
## BGP简介

在AS之间专门使用BGP(Border GatewayProtocol，边界网关协议)协议进行路由传递，相较于传统的IGP协议:
1. BGP基于TCP，只要能够建立TCP连接即可建立BGP。
	IGP 要求路径上的所有路由器都要开启 IGP，才可以互相建立邻居。BGP 中间经过的路由器没必要开，只要两端的开并且建立连接就可以。
2. 只传递路由信息，不会暴露AS内的拓扑信息。
3. 触发式更新，而不是进行周期性更新。

BGP是一种实现自治系统As之间的路由可达，并选择最佳路由的矢量性协议

- **BGP的特点:**
1. BGP使用TCP作为其传输层协议(端口号为179)，使用触发式路由更新，而不是周期性路由更新。
	1. **OSPF（IGP）**：邻居必须直连，中间每一台设备都跑 OSPF；跨网段的多台路由器两两建立邻接，整条链路所有节点纳入同一 IGP 域。
	2. **BGP（EGP）**：基于 TCP 单播会话，**中间转发设备完全不需要运行 BGP**，只要求两端 BGP 路由器 IP 互通即可建立邻居，中间设备只做普通 IP 转发。
2. BGP能够承载大批量的路由信息，能够支撑大规模网络。
3. BGP提供了丰富的路由策略，能够灵活进行路由选路和路由属性，并能指导对等体按策略。
4. BGP能够支撑MPLS/VPN的应用，传递客户VPN路由。
5. BGP提供了路由聚合和路由衰减功能用于防止路由振荡，通过这两项功能有效地提高了网络稳定性

- **BGP特征**
1. BGP使用TCP为传输层协议，TCP端口号179。路由器之间的BGP会话基于TCP连接而建立。
2. 运行BGP的路由器被称为BGP发言者(BGP Speaker)，或BGP路由器。
3. 两个建立BGP会话的路由器互为对等体(Peer)，BGP对等体之间交换BGP路由表。
4. BGP路由器只发送增量的BGP路由更新，或进行触发式更新(不会周期性更新)。
5. BGP能够承载大批量的路由前缀，可在大规模网络中应用。
6. 可以通过路径属性进行路径选择——重点
7. 矢量路由协议

- **EBGP，IBGP**
1. EBGP：位于不同自治系统的BGP路由器之间的BGP对等体关系。两台路由器之间要建立EBGP对等体关系，必须满足两个条件:
	1. 两个路由器所属As不同(即As号不同)。
	2. 在配置EBGP时，Peer命令所指定的对等体IP地址要求路由可达，并且TCP连接能够正确建立。
2. IBGP(Internal BGP):位于相同自治系统的BGP路由器之间的BGP邻接关系。

- 对等体关系建立
1. 双向的TCP连接三次握手，之后舍弃一条，使用一条。
2. open报文，协商BGP参数
	1. my AS：本端AS号
	2. hold time：默认180s，keepalive断开。
	3. BGP ID：自身router-id
3. keepalive：定期发送。

- 路由更新
1. 使用update通告，更新路由信息

- TCP 连接源地址：
1. IBGP使用loopback，冗余好，更稳定
2. EBGP默认使用物理接口地址——如果要用loopback，需要用connect-interface指定loopback接口

