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

## SPF
1. 根据1类和2类LSA计算最短路径。
2. 过程
	1. ![[Pasted image 20260810161855.png]]
	2. 

## I-SPF
1. 介绍
	1. 网络中节点
2. 触发LSA
	1. 1类路由更新
	2. 2类路由更新

### PRC

1. 介绍
	1. 网络上路由发生变化的时候，只对发生变化的路由进行重新计算
	2. PRC不计算节点路径，而是根据SPF算出来的最短路径树来更新内容
	3. PRC将路由（叶）放入整棵树中
2. 过程
	1. 某节点泛洪新增LSA
	2. 收到LSA创建新路由，继承原有访问节点的路径和下一跳，SPF不变，只在节点后面新增叶子。
3. 触发LSA
	1. 3类路由更新
	2. 5类路由更新
	3. 7类路由更新