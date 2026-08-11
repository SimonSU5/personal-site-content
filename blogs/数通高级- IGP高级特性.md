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

### SPF
1. 根据1类和2类LSA计算最短路径。
2. 过程
	1. 拓扑如下图![[Pasted image 20260810161855.png]]
	2. Dijkstra 核心集合定义
		- **S 集合**：已经确定最短路径的节点（已加入 SPF 树）
		- **T 集合**：临时候选、最短距离待定节点
	3. 初始状态
		S = {R0}
		距离表：
		![[Pasted image 20260810162125.png]]
	4. **遍历 R0 所有邻居**
		R0→R1 cost=10；R0→R2 cost=20
		更新距离
		- dist(R1)=10
		- dist(R2)=20
		T={R1 (10), R2 (20)}
		挑选 T 开销最小：**R1(10)**
		R1 移入 S，S={R0,R1}
	5. **遍历 R1 邻居**
		邻居：R0 (已经在 S)、R3，R1‑R3 cost=5
		dist (R3)=dist (R1)+5 = 10+5 =15
		T={R2 (20),R3 (15)}
		挑选最小 R3 (15)
		R3 加入 S，S={R0,R1,R3}
	6. **遍历 R3 邻居**
		R1 (S 内)、R2；R3‑R2 cost=1
		dist (R2)= min (旧值 20 , dist (R3)+1=16) → 更新 dist (R2)=1
		T={R2 (16)}
		取出 R2 加入 S
		S={R0,R1,R3,R2}

### I-SPF
1. 介绍
	1. 网络中节点断开，导致树需要重新计算时触发
	2. 断开后，下方子树剥离SPF
	3. 对所有涉及变动的子拓扑重新计算局部SPF
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

### 智能定时器
1. 路由拓扑/链路快速震荡时，响应过快会导致路由反复震荡——智能定时器会减少突发时间的快速响应，避免过度占用CPU。
2. 控制LSA的生成和接收
	1. 同一条LSA 1s内不再生成。
	2. LSA接收时间间隔为1s。
	3. **对于需要快速收敛的网络，可以配置间隔时间为0**
	4. **基础配置命令——OSPF协议视图下**
		1. lsa-originate-interval {0 | {intelligent-timer *max-interval start-interval hold-interval*}}
		2. 初次更新间隔时间为start-interval（缺省500ms）
		3. n次更新的LSA间隔时间为hold-interval（缺省1000ms） * 2 ^ (n-2) 
		4. 当上步反复执行到最长间隔max-interval（缺省5000ms），OSPF连续三次更新LSA的间隔时间都是最长时间，之后再次返回步骤1，按照初始时间间隔更新LSA。
3. 控制路由计算
	1. 计算最短路径（SPF和I-SPF）的时间可以由智能定时器指定间隔。
	2. **基础配置命令——OSPF协议视图下**
		1. spf-schedule-interval {*interval1* | intelligent-timer *max-interval start-interval hold-interval* | millisecond *interval2*}
		2. 初次计算SPF间隔时间为start-interval（缺省500ms）
		3. n次计算SPF间隔时间为hold-interval（缺省1000ms） * 2 ^ (n-2) 
		4. 当上步反复执行到最长间隔max-interval（缺省10000ms），OSPF连续三次计算SPF的间隔时间都是最长时间，之后再次返回步骤1，按照初始时间间隔计算SPF。

### FRR

1. LFA算法算出最短无环备份路径，50ms内切换
2. 