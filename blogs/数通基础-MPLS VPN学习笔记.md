---
title: MPLS VPN学习笔记
excerpt: MPLS VPN学习笔记
category: 学习笔记
tags:
  - 路由协议
  - 数通
  - 学习笔记
  - 
cover: assets/covers/image.jpg
date: 2026-07-24
published: false
---
基本概念
1. MPLS位于2层和3层之间
2. 通过数据链路层和网络层之间增加的额外MPLS头部，实现MPLS的快速转发。
3. MPLS域
	1. 一系列运行MPLS的网络设备构成了一个MPLS域
	2. LSR：运行了MPLS的路由器叫做LSR
	3. LER：位于网络边缘的路由器 edge
	4. 核心LSR：位于区域内部的为核心LSR（所有接口都运行MPLS）
4. 流量方向分类
	1. 入站LSR：压入MPLS头部
	2. 中转LSR：只看MPLS头部，对报文进行例如**标签置换**操作，然后进行转发
	3. 出站LSR：移除MPLS头部，还原回IP网络。
5. FEC 转发等价类（forwarding equivalent class）
	1. 是数据流，这类数据流会被以同样的方式处理。
	2. 可以基于多种方式划分，最常见是目标网段。还有其他的如IP优先级等
	3. 数据属于哪个LSP，由数据进入MPLS域时的入站LSR决定
	4. MPLS标签通常和FEC相对应。必须有某种机制使网络中的LSR获得关于某FEC的标签信息。
6. LSP 标签交换路径（label switched path）——隧道
	1. 标签报文穿越MPLS网络时走的路径
	2. 同一个FEC报文通常采用相同的LSP穿越MPLS域。对于同一个FEC（比如同一个目标网段），MPLS LSR总以相同的标签转发。
7. MPLS 标签——可以有多个标签
	1. label：20bit——类比目标IP地址
	2. EXP：用于CoS，长度3bit。
	3. S：栈底位：表示是最后一个标签
	4. 