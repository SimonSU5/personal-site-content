---
title: OSPF学习笔记03——协商和选举过程
excerpt: OSPF学习笔记03——协商和选举过程
category: 学习笔记
tags:
  - 路由协议
  - IGP
  - 数通
  - 学习笔记
cover: assets/covers/OSPF学习笔记-封面.png
date: 2026-07-09
published: false
---
## OSPF报文格式和类型

OSPF报文工作在IP层，协议号89

| 类型  | 报文名称                 | 报文功能        |
| --- | -------------------- | ----------- |
| 1   | hello                | 发现和维护邻居关系   |
| 2   | database description | 交互链路状态数据库摘要 |
| 3   | link state request   | 请求特定的链路状态信息 |
| 4   | link state update    | 发送详细的链路状态信息 |
| 5   | link state ack       | 确认LSA       |

![[Pasted image 20260709121244.png]]

## OSPF协商过程

1. **邻居关系建立**
	1. 邻居建立
		1. A到B，声明自己是A，邻居是空
		2. B到A，声明自己是B，邻居是空
		3. A到B，声明自己是A，邻居是B
		4. B到A，声明自己是B，邻居是A
		5. DR（指定路由器）和BDR（备份指定路由器）选举
	2. 邻居状态
		1. down——表示目前没有邻居（对应邻居建立的第1步之前）
		2. init——已经收到对面的hello报文，但是对面的邻居不是我（对应邻居建立的第1，2步）
		3. 2-way——收到对面的hello报文，而且对面的邻居已经是我（对应邻居建立的第3，4步）
	3. hello头
		1. version：OSPF版本
		2. 源OSPFrouter：源路由器router ID
		3. area ID：源路由器area ID
		4. 认证相关报文
	4. hello报文
		1. network mask——掩码不同，邻居关系无法建立
		2. hello interval——hello报文发送间隔，一般为10秒
		3. routerDeadInterval——hello报文失效时间，一般为40秒。40秒未收到hello报文，邻居断开。
		4. neighbor——邻居router id，用于邻居建立3，4步标识已获取邻居信息。
		5. router priority——DR优先级，默认为1。如果设置为0，则路由器不能参与DR或者BDR选举。
	5. 邻居保持
		1. 每40秒（hello interval时长）发一次hello报文，维持邻居关系。

2. **邻接关系建立**
	1. 协商主从关系建立（保证DD报文的可靠性，发送LSDB摘要）
		1. 主从关系建立过程
			1. R1发送DD报文（seq=X，I=1，M=1，MS=1）
				1. seq是报文序列号，防止乱序
				2. I（init），代表是最先发送的DD报文
				3. M（more），代表后面还有更多的DD报文
				4. MS（master/slave），1代表master，说明发送方自认为自己是master。通过一轮交互后，会根据router id的大小选举master，router id大的为master。
			2. R2发送DD报文（seq=Y，I=1，M=1，MS=1）
		2. 邻居状态
			1. 2-way——邻接关系建立前，为双向邻居关系
			2. Exstart——开始发送DD报文
			3. Exchange——开始发送LSDB摘要
	
	2. LSDB摘要交互（通过DD报文实现）
		1. LSDB摘要交互过程
			1. R1向R2发送本机LSDB摘要（目录）（seq=Y，LSDB摘要）
				1. R1比较router ID，发现R2的router ID更大。因此判定R2为主router。seq取Y
			2. R2向R1发送本机LSDB摘要（seq=Y+1，LSDB摘要，MS=1）
				1. R2（主设备）一定要收到从设备的DD后，才可以继续发送。这也就保证了可靠性。
			3. R1向R2发送DD报文，确认收到LSDB摘要（seq=Y+1）
		2. LSDB摘要交互状态
			1. Exchange——开始发送LSDB摘要
			2. Loading——开始发送LSR，LSU，LSA报文
		3. LSDB摘要内容：
			1. 本机的LSDB有哪些目的地址（段）的路由
	
	3. LSDB更新
		1. LSDB更新过程
			1. DD中如果包含自己LSDB没有的路由，就会发送LSR请求LSA完整信息。
			2. 对方发送LSU报文，告诉我路由详细信息。
			3. 我发送LSA报文，表示我收到。（未收到，则超时重传LSU）
		2. LSDB更新状态
			1. loading——开始发送LSR，LSU，LSA报文
			2. full——表示已经完成了和邻居的LSDB同步
		**完成同步后，同一个区域内的所有路由器拥有完全一致的LSDB。**

3. 双方交互完成，各自计算OSPF路由表，更新路由表

## DR和BDR作用

- MA网络（广播型网络，基于以太网）中才会使用，并且必须使用（不能把优先级全都调整到0）。
- 在2-way状态下直接选举

1. MA网络的问题
	1. 邻接关系过多，一个广播域中存在n台OSPF的路由器（星型组网情况下），将会有Cn2 = n * (n-1)/2 个邻接关系。
	2. 重复的LSA泛洪，资源浪费
2. 解决办法——在MA网络中选举DR
	1. DR指定路由器负责在网络中建立和维护邻接关系，并负责LSA的同步
	2. DR和其他所有路由器建立邻接关系，其他路由器之间不直接交换路由链路状态信息。
	3. BDR作为DR的备份，当DR下线，BDR立刻接收DR工作。
3. 选举原则
	1. 非抢占式，选好后不再选择新DR。DR固定，新加入的路由器就算优先级更高，也不再选举为DR
	2. 基于接口选举
		1. 接口DR优先级越大越优先
		2. 接口DR优先级相等时，router id越大越优先
4. DR/BDR/DRother状态
	1. DR/BDR与DRother之间为full状态
	2. DRother之间为2-way状态
5. DR/BDR/DRother组播组
	1. DR/BDR为224.0.0.6，目的地址为224.0.0.6的会被DR/BDR接收
	2. DRother为224.0.0.5，目的地址为224.0.0.5的会被DRothre接收
	3. DRother->224.0.0.6->DR/BDR接收->224.0.0.5->DRother接收
6. 需要选举DR的情况：
	1. 以太网链路和帧中继链路（现在已经很少用了）
	2. 可以在华为接口中用ospf interface {p2p I p2mp | broadcast | nbma }更改类型，关闭选举或开启选举（默认开启）

| OSPF网络类型          | 链路层协议    | 是否选举DR | 是否和邻居建立邻接关系                                              | 命令                       |
| ----------------- | -------- | ------ | -------------------------------------------------------- | ------------------------ |
| 点到点point to point | PPP，HDLC | 否      | 是                                                        | ospf interface p2p       |
| 广播broadcast       | 以太网      | **是**  | DR与BDR、DRother建立邻接关系BDR与DR、DRother建立邻接关系DRother之间只建立邻居关系 | ospf interface broadcast |
| 帧中继NBMA           | 帧中继      | **是**  | DR与BDR、DRother建立邻接关系BDR与DR、DRother建立邻接关系DRother之间只建立邻居关系 | ospf interface nbma      |
| P2MP              | 手工指定     | 否      | 是                                                        | ospf interface p2mp      |

## DR和BDR选举实验
![[Pasted image 20260709180919.png]]
1. 新配交换机等40s如果没有hello的回应（其余交换机配置较慢情况下）则直接成为DR。
2. 四个交换机配置后并重启，则按照上面的原则选举DR（接口优先级，如果相同则按照router id）。

### 重启之前
![[Pasted image 20260709183148.png]]
![[Pasted image 20260709183202.png]]

### 重启之后
![[Pasted image 20260709184049.png]]
重新选举了，正常应该按照router id选举DR（我没有进到int配置ospf dr-priority，所以四台都是默认值1），结果应该是R4为DR。但是现在是R3为DR，因此应该是R3先开机了，被选举成了DR。