---
title: OSPF学习笔记05——防环
excerpt: OSPF学习笔记05——防环
category: 学习笔记
tags:
  - 路由协议
  - 数通
  - IGP
  - 学习笔记
cover: assets/covers/OSPF学习笔记-封面.png
date: 2026-07-15
published: false
---
# 区域间防环机制

1. 要求OSPF非骨干区域只能和骨干区域相连
	1. 非骨干区域只能和骨干区域相连，形成**类似**星型组网（并不是真正的星型组网，因此还存在环路可能）
	2. 只有非骨干区域和骨干区域链接的路由器才能作为ABR，才能产生和转发3类LSA。这样3类LSA在f