---
title: OSPF学习笔记09——OSPFv3
excerpt: OSPF学习笔记09——OSPFv3
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
| LSA 类型 | LSA 名称                                 | 始发路由器（Origin Router）                 | 是否携带 IPv6 前缀                  | 泛洪范围                                 | OSPFv2 对等 LSA               | 类比                    |
| ------ | -------------------------------------- | ------------------------------------ | ----------------------------- | ------------------------------------ | --------------------------- | --------------------- |
| Type 1 | Router-LSA<br><br>路由器 LSA              | 域内每一台运行 OSPFv3 的路由器（本机生成）            | ❌ **不带前缀，只描述拓扑、链路、Metric、邻居** | 整个区域（Area scope）                     | Type1 Router-LSA            | 1+9可以泛洪所有的地址段         |
| Type 2 | Network-LSA<br><br>广播网络 LSA            | 广播 / NBMA 网段的 DR                     | ❌ **不带前缀，只列出本网段所有邻接路由器**      | 整个区域（Area scope）                     | Type2 Network-LSA           |                       |
| Type 3 | Inter-Area-Prefix-LSA<br><br>域间前缀 LSA  | **仅 ABR 生成**                         | ✅ 携带跨区域 IPv6 前缀               | 整个区域（Area scope）                     | Type3 Network-Summary-LSA   |                       |
| Type 4 | Inter-Area-Router-LSA<br><br>域间路由器 LSA | **仅 ABR 生成**                         | ❌ 不携带网段前缀，只通告 ASBR 位置         | 整个区域（Area scope）                     | Type4 ASBR-Summary-LSA      |                       |
| Type 5 | AS-External-LSA<br><br>AS 外部 LSA       | **仅 ASBR 生成**                        | ✅ 引入外部 IPv6 前缀                | 整个 AS（AS scope，除 Stub/NSSA）          | Type5 AS-External-LSA       |                       |
| Type 7 | NSSA External-LSA<br><br>NSSA 外部 LSA   | **仅 NSSA 区域内 ASBR 生成**               | ✅ NSSA 引入的外部 IPv6 前缀          | 仅限本 NSSA 区域（Area scope）              | Type7 NSSA External-LSA     |                       |
| Type 8 | Link-LSA<br><br>链路 LSA                 | **每个 OSPFv3 接口本机独立生成**（一接口一张）        | ✅ 携带该链路本地接口前缀                 | **链路本地范围 Link-local scope（不能跨网段转发）** | v3 独有，v2 无对应                | 有点像ARP的作用，向邻居通告直连v6地址 |
| Type 9 | Intra-Area-Prefix-LSA<br><br>域内前缀 LSA  | 产生对应 Type1/Type2 的同一台路由器（普通路由器 / DR） | ✅ 携带域内 IPv6 网段前缀              | 整个区域（Area scope）                     | v3 独有，承担 v2 Type1/2 携带网段的工作 |                       |
![[2363ede4b675d391763e69793aca5793.png]]
1. R1 1口和R4相连，应该选择R4 router id。
2. network LSA 由area 0 DR发出，缺省情况下选择router id大的，应由R3发出。应该选择R3 router id。
3. inter-area-prefix-LSA由ABR发出，选择R1 router id。
4. intra-area-prefix-LSA由area 0 DR发出，应该选择R3 router id。