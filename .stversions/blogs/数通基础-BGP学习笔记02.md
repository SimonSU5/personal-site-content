---
title: BGP学习笔记02
excerpt: BGP学习笔记02
category: 学习笔记
tags:
  - 路由协议
  - 数通
  - BGP
  - 学习笔记
cover: assets/covers/BGP学习笔记-封面.png
date: 2026-07-21
published: false
---
1. EBGP配置
	1. 使用物理接口
		1. 默认使用物理接口
		2. 默认使用TTL=1
		3. peer 地址 as-number 对端as
	2. 使用环回口
		1. 静态路由添加环回口——跨AS无法使用IGP协议
		2. peer 地址 as-number 对端as
		3. 指定环回口：peer 环回口地址 connect-interface LoopBack x
		4. TTL改为2：peer 环回口地址 ebgp-max-hop 2
2. IBGP配置
	1. 使用物理接口
		1. 默认使用物理接口
		2. 默认使用TTL=256
		3. peer 地址 as-number 对端as
	2. 使用环回口
		1. OSPF协议添加环回口路由
		2. peer 地址 as-number 对端as
		3. 指定环回口：peer 环回口地址 connect-interface LoopBack x
	3. 使用环回口更好，因为网络冗余的情况下，端口关闭还是可以通过冗余路由建立邻居。


---
BGP报文

1. 应用层报文：BGP公共报文头+BGP packet
	1. 公共报文头
		1. marker 16B
			1. 所有16B都为1，表明BGP的明确开头。
		2. length 2B
		3. Type 1B
			1. packet类型（上面的Type）
				1. open：协商BGP对等体参数，建立对等体关系。在tcp链接之后
				2. update：发送BGP路由更新。在peer建立成功或者有改变
				3. notification：报告错误信息。在运行过程中发现错误时，通告对等体中断peer
				4. keepalive：维护对等体关系
				5. route-refresh：路由策略发生变化时，触发请求对等体重新通告路由
	2. packet内容
		1. open
			1. version（8bit）v4或v6
			2. my as（16bit）
			3. hold time（16bit）默认180s，向下协商
			4. bgp ID（32bit）router id，以IP的形式表示
			5. opt param len
				1. optional param
		2. update——发布或者撤销路由
			1. 一个update可以包括属性一致的多个路由
			2. 一个update可以包括多个不可达路由，告诉对等体撤销
			3. 组成
				1. withdrawn routes：不可达路由
				2. path attributes：路径属性
				3. NLRI：可达路由的前缀和前缀二元组——路由信息
		3. notification
			1. error code
			2. error subcode
		4. keepalive
			1. 没有packet内容，只有公共头
		5. route refresh
			1. AFI：地址族标识16bit（ipv4或者ipv6）
			2. res：保留 8bit置0
			3. SAFI：子地址族标识8bit

---

BGP邻居状态机

1. 状态机：
	1. Idle：开始准备——无路由
	2. connect：正在进行TCP链接，等待完成中。**认证**——有路由，但是对方无路由
	3. active：TCP链接失败，反复尝试TCP链接——有路由，但是对方无路由/没有指定loo0接口；as号配错了
	4. opensent：tcp已经建立成功，开始发送open 包。
	5. openconfirm：参数能力特性协商成功，已发送keepalive。等待对端发送keepalive
	6. established：已经收到对方的keepalive包，双方协商一致。开始用update通告路由。
	7. **任何一个环节错误，都会发送notification报文，返回idle状态**

---
BGP路由生成

1. 被动注入，主动不会生成。
2. 注入方式
	1. network
		1. 直接宣告网段
		2. * 代表有效路由  > 代表最优路由
		3. 必须时有效最优路由才可被通告到peer
	2. import-route
		1. import-route ospf 1（进程号）
		2. 所有ospf都会被引入
		3. 可以配合过滤手段减少
	3. 路由聚合
		1. 本端聚合
		2. 首先保证本端BGP有明细路由
		3. bgp进程下 aggregate xxxx x detail-suppressed
		4. 如果没有detail-suppressed，本端路由会有所有明细最优有效路由和聚合最优有效路由。有抑制明细的情况下，>会转变为s，代表被聚合。
3. 注入流程
	1. IGP路由表
	2. 注入BGP路由表

---
BGP 通告原则

1. 只发布最优有效路由
	1. 有效路由：下一跳地址可达（不是所有情况）
	2. 最优路由：[[数通基础-BGP学习笔记02#BGP选举原则]]
2. 从EBGP对等体获得的路由，会发布给所有对等体
	1. 可以给IBGP也可以给EBGP
3. 从IBGP对等体获取的路由不会发布给IBGP对等体
	1. 防止环路
	2. 可能会导致学不到路由
		1. 可以做IBGP全互联，两两建立邻居关系
4. 当IBGP从自己的IBGP学习到一条路由时，不会把这条路由通告给EBGP对等体。防止路由黑洞。除非自己的IGP（包含静态路由）要求IBGP学习。（华为默认不开）
	1. IBGP并非直连，可能会造成路由黑洞。R4-ebgp-R2-R1-R3-ebgp-R5 R2和R3为IBGP邻居。R4发布到R2，R2发布给R3，R3发布给R5。R5访问，R3下一跳为R2，但是R3到R2要过R1，R1没有路由，造成路由黑洞。**——这种情况下看dis bgp routing-table 会发现没有 * 。因为下一跳不可达**
	2. 华为默认不开，因为BGP40W+条>>IGP承受条数，所以大型网络下大部分路由都学不了。
	3. 解决办法是使用全互联，让R1也学到路由。

---
实验
![[Pasted image 20260725131000.png]]
1. IGP使用OSPF，R1和R4建立了IBGP邻居，R1和R5通过EBGP建立邻居，R4和R6通过EBGP建立了邻居。
	1. R5使用EBGP通告了一个环回口地址5.5.5.5。R1将收到这个通告并且发给IBGP邻居4.4.4.4。
	2. 4.4.4.4收到5.5.5.5路由，下一跳是R5的0/0/0接口地址。
	3. 4.4.4.4不认识R5的接口，因此路由表显示无效 下一跳不可达。因此4.4.4.4不会将路由给R6。
	4. R1需要增加命令：BGP进程下，peer 4.4.4.4 **next-hop-local**，下一跳将会变为1.1.1.1
	5. 此时4.4.4.4的5.5.5.5路由下一跳为1.1.1.1。路由可达，路由表中此条路由有效。通告给R6。
	6. 此时4.4.4.4依然无法ping通5.5.5.5。因为下一跳是1.1.1.1，会把5.5.5.5报文给R2。R2没有5.5.5.5的路由，因此黑洞。
	7. **如果R6通告6.6.6.6到EBGP，这时候R4不会将这个路由作为最优有效路由。因为6.6.6.6是peer地址。**
	8. 路由黑洞可以用ospf引入bgp的方式引入。（不可取，没有做路由过滤的时候，会使BGP的所有路由都引入，会太大）也可以用全互联（推荐）
2. 全连接建立，每个IBGP都有peer建立，指定环回口，next hop local。每个EBGP都有peer建立（如果环回口的话需要指定环回口，ebgp-max-hop 2）。
	1. R5 R6各宣告了一个网段（使用环回口模拟）
	2. R5 ping -a R5环回口地址 R6环回口 通。（注意一定要指定源地址。）

---
BGP路径属性

1. 每一条路由都有多个路径属性
	1. 公认属性——每台BGP路由器都能识别
		1. 公认必遵——必须包括在update报文中
			
			- **重要：如果在BGP做了route policy，需要在后面加一个全放通的policy。不然acl以外的路由会被丢弃**
			
			1. **origin**
				1. 标记BGP路由的起源
				2. 类型
					1. IGP（i）如果是使用network引入
					2. EGP（e）如果通过EGP学习到（几乎不用了）
					3. incomplete（？）如果是通过其他方式，如import-route
				3. 当去往同一个目的网络存在多条不同origin属性的路由时，BGP将采用IGP>EGP>incomplete的方式选路。
				4. acl+（route-policy if-match+apply origin xx）+peer 指定route polocy
			2. **as_path**
				1. 是前往目标网络的路由经过的as号列表，新加都加在左边
				2. 作用
					1. 防止环路
				3. 只有通过EBGP的时候才会加上AS号
				4. 影响路由优选的原则
					1. as-path更少的优选
					2. 可能as-path更多的带宽更大。可以对as path进行修改。
					3. 修改AS-path
						1. additive（最常用）：在左侧加上 apply as-path 300 additive
						2. overwrite：将之前所有的as-path都去掉，加上新的。 apply as-path 400 overwrite
						3. none overwrite：将之前所有的as-path都去掉。apply as-path none overwrite
						4. 总体思路：acl+（route-policy if-match+apply）+peer 指定route polocy
				5. as-path 类型
					1. as-sequence：有序列表，默认
					2. as-set：默认情况下，聚合之后会丢失前面的所有as。设置as-set后，聚合后，会将之前的所有as变成聚合，作为上一个as列表。**需要在聚合后添加as-set关键字。**
			3. **next_hop**
				1. 用于指定下一跳地址
				2. 当路由器学习到一条路由后，会对下一跳检查。如果不包括在当前路由表内，则不可达。
				3. next hop缺省操作
					1. EBGP：通告路由时会使用peer本端地址。
					2. IBGP：通告路由给IBGP邻居时，下一跳不变。【会导致不可达，所以需要使用peer xxxx next_hop_local】
					3. EBGP特殊情况：如果有一个路由条目EBGP的peer和下一跳的地址在同一个网段，则下一跳不变直接传给EBGPpeer，这样对端的EBGPpeer可以直接给本设备的下一跳。				 ![[Pasted image 20260725164827.png]]
		2. 公认任意——可能包括在update报文中
			1. **local preference 本地优先级** 
				1. 针对AS内部，告诉AS路由器，哪条路经是离开as的首选路径
				2. local_preference属性越大路由越优。缺省为100
				3. 只传给IBGP对等体
				4. **必须在import方向上**修改local_preference属性
			2. atomic_aggregate 原子聚合 
				1. 运行了aggregate xxxx xx detail_suppressed就会携带，告诉接下来的对等体不要再做聚合了。已经丢失了明细
	2. 可选属性
		1. 可选过渡——可以不识别，但是会通告给其他对等体
			1. aggregator 聚合者
				1. 运行了aggregate xxxx xx detail_suppressed就会携带
			2. **community 团体属性**
				1. 打路由标记，为不同路由进行归类，**并且跨AS传递**。简化路由策略执行。（不需要用ACL的路由前缀，只需要团体属性即可）
				2. 属性格式：AA:NN，各16bit，AA推荐为AS号，NN为自定义。
				3. 公认属性：
					1. 0（internet）：收到此团体属性后，可以向任何BGP发送
					2. no_advertise（0xFFFFFF02）：收到此团体属性后，不向任何BGP对等体发送
					3. No_export（0xFFFFFF01）：收到此团体属性后，不想AS外部发送。**也就是第一次会发到下一个AS，然后收到这个团体属性的路由的AS不再转给其他AS。**
				4. 更改属性
					1. acl+（route-policy ifmach+apply comm xxx）+peer xxxx advertise-community
					2. **给所有其他IGP发送的时候，也要加peer xxxx advertise-community**必须加，否则会丢community属性。
		2. 可选非过渡——可以不识别，也不会通告给其他对等体
			1. **MED值 类似于开销值** 多出口鉴别器
				1. 影响外部对等体流量如何进入对等体
				2. MED值越小，路由越优先
				3. MED被传递给EBGP邻居后，对等体可以在AS内传递。但是再次传递的时候就不会携带。
				4. 缺省情况下，路由器只比较来自**同一相邻**BGP路由的MED值，如果是不同的AS，则不比较
				5. 是否携带MED值的情况：
					1. 本地始发，network或者import-route 引入。则会携带
					2. 从ebgp收到，则出AS时不会携带。
					3. IBGP对等体之间传递时，会保留并传递。
				6. MED默认操作
					1. IBGP network或者import-route 引入，MED会继承IGP的metric值。
					2. BGP路由器将本地直连、静态路由通过network或者import-route 引入，MED为0。
					3. MED不会跨AS传递。
					![[Pasted image 20260725215923.png]]
					4. 在路由器上运行default med可以更改med值。
			2. cluster-list 路由反射器簇列表
			3. originator-ID 路由反射器起源id

---
IBGP路由反射器

1. 引入路由反射器后，存在角色：
	1. RR：路由反射器
	2. client：路由反射器客户端（本身不知道自己是客户端，无需配置）
2. 路由反射规则（非非不传，其他都传）
	1. 从自己的非客户对等体学习到IBGP路由，则可以反射给所有客户。但是不传给非客户端。
	2. 从自己的客户学习到一条IBGP路由，则可以反射给所有非客户以及所有除了发送客户以外的所有客户
3. 设计路由反射器时需要考虑防环
	1. cluster-list 路由反射器簇列表
		1. 路由反射簇包括反射器RR和Client。一个AS中可以有多个反射簇。
		2. 
	2. originator-ID 路由反射器起源id
		1. 路由环路时，发送者会丢弃收到的含有自己originator-id的路由。
		![[Pasted image 20260725222843.png]]