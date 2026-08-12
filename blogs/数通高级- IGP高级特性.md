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
## OSPF

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
2. 链路保护不等式1：备份路由器到目的的cost要小于从主路由到目的地那条路径的cost
3. 节点链路双保护不等式1：备份路由器到目的地的cost要小于从主路由到目的地那条路径的cost，目的地的cost要小于从备份节点到目的地那条路径的cost
4. 配置
	1. ospf视图下，frr进入frr视图
	2. loop-free-alternate
	3. 接口视图下可以禁止frr 接口视图下ospf frr block：主要为了不让这个接口上的链路成为备份链路，重要业务不会被挤占

### BFD-OSPF联动
1. 缺省情况下，OSPF探测故障通过hello超时。
2. 过程
	1. OSPF邻居FULL后，通知BFD建立会话
	2. 链路down后，BFD直接通知OSPF peer down 毫秒级响应
3. 配置
	1. OSPF视图下，bfd all-interfaces enable
	2. 接口视图下，ospf bfd enable


### OSPF路由控制
1. 等价路由
	1. 路由表中存在同一目的地址开销值相同，则依据负载分担的方式从多条路由发送到目的地址
	2. 负载分担方式——逐包/逐流
	3. 命令——配置最大的负载分担条数（默认8条）
		1. ospf视图下  maximum load-balancing *number*
2. 缺省路由
	1. OSPF 3类和7类缺省路由——stub和NSSA区域
	2. import-route无法引入缺省路由，只能使用default-route-advertise
	3. 再通告缺省路由的时候，必须检查缺省路由的优先级（比如静态路由优先级高）
3. LSA过滤
	1. 特定LSA过滤
	2. 命令：ospf filter-lsa-out xxx（all，summery，ase，nssa等，都可以应用acl）
	3. **对于已经发送的LSA，只能等待老化时间**——解决办法只能是重建OSPF
4. OSPF database overflow
	1. 设置路由器非缺省外部路由数量上限
	2. 过程
		1. 进入overflow状态
			1. 路由器删除**所有**自己产生的非缺省外部路由
			2. 启动overflow定时器（默认5s）
		2. 处于overflow状态
			1. 不产生非缺省外部路由
			2. 丢弃所有非缺省外部路由LSA
			3. 定时器超时，自动检查非缺省外部路由数量是否还超过上限
				1. N：退出overflow状态
				2. Y：重启overflow计时器
			4. 退出overflow状态
				1. 删除overflow定时器
				2. 产生非缺省外部路由
				3. 接收非缺省外部路由并能回复LSA
				4. 准备下次进入overflow状态

### OSPF多进程
1. 适用于MPLS-VPN
2. 公司网络合并，有两个area 0，并且area 0无法直接对接。

### OSPF和BGP联动
情景：
![[Pasted image 20260812150626.png]]
1. 步骤
	1. 当R2重启，IBGP不会down，流量瞬间切换到R1-R3-R4。
	2. R2启动后，OSPF建立但是IBGP还未收敛，此时R1会选择R2转发，但是R2没有BGP路由表，导致丢包。
	3. 在OSPF收敛但是BGP没有收敛的时候，R2要配置stub-router on-startup [*interval* ] 或者使用 wait-for-bgp。在重启的时候会通告cost为65535来避免成为数据转发路由器


### Forwarding Address
1. 5类 7类 LSA特有
2. 目的：防止某些特殊场景下的次优路径
3. 场景1——5类FA作用：
	![[Pasted image 20260812151451.png]]
	1. 在同一广播域内，R2引入了外部路由下一跳R1，R3就会先把下一跳设置为R2，由R2转到R1——次优路径
	2. FA直接指定R1，防止次优路径
	3. 当FA非0 ，会直接计算到FA地址的下一跳，规避次优路径
	4. FA取值——当满足以下条件才可以非零
		1. ASBR链接外部路由的接口必须启用OSPF
		2. 该接口没有启用siilence-interface
		3. 该接口必须为broadcast/NBMA
		4. 该接口的IP地址必须在network声明内
		5. FA地址被置为外部LSA的下一跳地址。
4. 场景2——7类LSA场景：
	![[Pasted image 20260812160729.png]]
	1. 7转5 路由器如果处于低带宽链路，如果没有FA地址，就会认为只能通过R3，流量就被引导到低带宽链路
	2. 使用FA后，会认为此NSSA的网段必须经过R5，即会自动找R5的路由，流量就会被引导到高带宽链路。

### OSPF GR——graceful restart（一种NSF 不间断forwarding的策略）

1. 保证重启过程中还能进行数据转发，控制层面不影响数据转发。
2. 新增type 9 opaque LSA

### NSR 不间断路由

1. 目的：设备倒换过程中，路由处理不中断
	1. 邻居和拓扑信息不丢失
	2. 邻居关系不中断
	3. 收敛速度比NSF快
	4. 不依赖对端设备
2. 场景：在主用和备用控制板都存在的时候，主用控制板故障不影响邻居关系的技术
3. 过程
	1. 批量备份：备板重启，主板批量备份到背板
	2. 实时备份：备板实时备份主板的控制平面和转发平面——当邻居或路由发生变化时，主控板会实时将信息备份到备板
	3. 主备倒换：硬件探测，备板直接升为主控板，此时倒换很快完成，邻居不会断开。
	4. 主备不抢占，老主会成为新备

### 实验
![[Pasted image 20260812141226.png]]
1. 要控制外部路由出口只走一边：
	1. type 2引入，忽略外部的cost
	2. 指定cost，让边界1的cost比边界2引入的cost小。边界1down后自动切到边界2。
	3. **注意import时的cost 和 policy的cost 区别！**
		1. import时候使用的type 2 cost 100会将所有import的类型都改成cost 100。
		2. 这里需要使用route-policy来指定某条静态路由cost 100，另一条静态路由cost 200。所以一定要用route-policy来指定。
2. 内部路由控制
	1. 使用cost设计内部路由控制。

### OSPF总结
import xxx filter-policy：调用acl，import 的时候放通/禁止路由
ospf视图下 filter-policy：调用ACL，在import/export方向上放通/禁止路由
route-policy：调用acl+if-match+apply，import的时候可以更改路由属性
cost：cost是接口属性

## ISIS

### LSP快速扩散
1. 正常情况下，收到LSP后会更新LSDB，之后再用一个定时器将LSP扩散
2. 快速扩散特性可以在更新LSDB之前就把**一定数量的**LSP扩散出去，这个数量可以配置。

### 路由控制
1. ISIS优先级
2. 接口开销：注意窄度量0-63，宽度量能到4个字节。网络大需要改到宽度量
3. 路由渗透
4. 等价路由
	1. 可以配置最大等价路由条数
	2. 通过nexthop xxxx weight *weight* 控制，weight越小越优先
5. 缺省路由通告
6. 引入外部路由
7. filter-policy

### 分片扩展
1. 标准分片：分片ID8bit，最多256片
2. 分片扩展：配置虚拟系统ID（router id），最多产生50个
	1. 初始系统：真实系统
	2. 系统id：初始系统id
	3. 虚拟系统、附加系统id：生成扩展LSP分片
	4. 24号TLV：表示初始系统和虚拟系统的关系

### ISIS GR
1. 211号TLV
