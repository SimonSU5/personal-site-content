---
title: STP学习笔记
excerpt: STP学习笔记
category: 学习笔记
tags:
  - 路由协议
  - 数通
  - 学习笔记
  - 
cover: assets/covers/image.jpg
date: 2026-07-28
published: false
---
1. 环路网络造成的问题
	1. 广播风暴：指定广播，未知单播，组播帧
	2. mac漂移
2. 生成树作用
	1. 破环
	2. 备份

---
STP
1. 根桥
	1. 桥ID（BID）：优先级.MAC 优先级越小越优，MAC越小越优
2. 开销
	1. 带宽越大，开销越小
	2. 根路径开销（RPC）：到根桥的开销总额
3. 端口ID
	1. 优先级.接口编号
	2. 优先级默认128
	3. 接口编号为接口的名称
4. BPDU
	1. 配置BPDU
		1. 桥ID
		2. 接口ID
	2. TCN BPDU