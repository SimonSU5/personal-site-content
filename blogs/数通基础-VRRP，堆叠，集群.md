---
title: VRRP，堆叠，集群
excerpt: VRRP，堆叠，集群
category: 学习笔记
tags:
  - 数通
  - 学习笔记
cover: assets/covers/数通学习笔记-封面.png
date: 2026-08-04
published: true
---
## VRRP

> 用于解决单点故障问题

1. VRRP路由器
	1. 配置在接口上，通过接口工作
2. VRID
	1. 一个VRRP组，使用相同的VRID标识。
	2. 一个组内只能虚拟出一台设备
	3. 一个组内只能由一台Master设备
3. 虚拟IP地址
	1. 配置VRRP时手动指定
	2. 一般为网关地址
4. 虚拟MAC地址
	1. 自动生成，0000-5e00-01xx xx为VRID
5. 角色
	1. Master路由器
		1. 响应ARP，进行数据转发
		2. 一段时间内发送保活报文
	2. Backup路由器
		1. 实时侦听Master
		2. 未侦听到保活报文，则取代Master的工作
	3. 优先级
		1. 1-255，数值越大越优先，默认100
		2. 255表示IP拥有者——直接宣告自己时Master/真实IP地址=虚拟IP地址
		3. 值相等比较IP地址，越大越优先
6. 报文
	1. 目的地址
		1. 224.0.0.18
	2. ver
		1. v2适用于IPv4
		2. v3适用于IPv4和IPv6
	3. auth type
		1. 不认证：0
		2. 纯文本密码：1
		3. MD5：2
	4. adver interval：默认为1秒，发送保活间隔
7. 定时器
	1. adver interval
		1. 发送保活间隔
		2. 默认为1s
	2. master down定时器
		1. master down=skew-time + 3 * adver int
		2. skew-time=（256-priority）/256
		3. skew-time的用途就是抢占的时候剩余交换机还能够以优先级最高者获取Master位
8. 主备选举
	1. init+backup状态
	2. 经过master down间隔后，变为master
	3. 交换vrrp报文，比较优先级/IP地址
	4. Master发送免费ARP报文给所有链接设备
	5. 特殊情况
		1. 优先级255——直接切换成master
		2. 主动退出
			1. 发送优先级为0报文
			2. 没收到优先级为0报文情况下，等待master down时间
		3. 主备回切
			1. master出现故障，网络重新选举master
			2. 抢占模式（preempt，默认）：发现master优先级更低，会立即抢占
			3. 非抢占模式：保持backup状态，直到master状态失效。——主要为了不让IGP发生动荡
9. VRRP负载分担
	![[Pasted image 20260806180028.png]]
10. VRRP监视上行端口
	1. 主设备主动监视上行接口，当上行接口断开，主动降低优先级
11. 与BFD联动
	1. 通过联动，感知对端故障发生后，不再等待Master down计时器。