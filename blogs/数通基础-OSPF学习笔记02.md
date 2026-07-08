---
title: OSPF学习笔记02——实验
excerpt: OSPF学习笔记
category: 数通
tags:
  - 数通
  - 路由协议
  - IGP
cover: assets/covers/image.jpg
date: 2026-07-06
published: false
---
## 实验拓扑
![[Pasted image 20260708201848.png]]
## OSPF命令
![[Pasted image 20260708201912.png]]

- process ID用于隔离不通的工作域，不同的进程不能互相通告路由。
- router-id实际选举规则（首次配置时不按照笔记01中的选举规则）：
	- 