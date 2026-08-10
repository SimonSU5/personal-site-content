---
title: IGP高级特性
excerpt: IGP高级特性
category: 数通
tags:
  - 学习笔记
  - 
cover: assets/covers/image.jpg
date: 2026-08-10
published: false
---
## OSPF快速收敛

### PRC

1. 介绍
	1. 网络上路由发生变化的时候，只对发生变化的路由进行重新计算
	2. PRC不计算节点路径，而是根据SPF算出来的最短路径树来更新内容
	3. PRC将路由（叶）放入整棵树中
2. 过程
	1. 某节点泛洪新增LSA
	2. 收到LSA创建新路由，