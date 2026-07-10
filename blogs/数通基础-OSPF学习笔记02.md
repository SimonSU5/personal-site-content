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
	- 哪个接口的IP先配置，则选用哪个的（不一定时loopback地址）
	- 手动配置完后，要回到user view下reset ospf process让手动配置生效

![[Pasted image 20260708203744.png]]
![[Pasted image 20260708203915.png]]

- network 命令
	- 就是让接口在network后面的地址内的所有端口使能ospf，上面0/0/0和loopback地址在通告的两个地址段内，因此这两个端口使能ospf。
	- 也可以在接口中ospf enable，等效于ospf-area-0视图下的```network 接口地址``` 。

- 观察ospf运行状态命令
	- dis ospf peer [br] —— 查看OSPF邻居信息
	- dis ospf interface —— 查看OSPF运行接口
	- dead timer —— hello报文保活时间
![[Pasted image 20260709114724.png]]

- 观察OSPF lsdb
![[Pasted image 20260709115544.png]]

- 观察OSPF路由表
![[Pasted image 20260709115556.png]]