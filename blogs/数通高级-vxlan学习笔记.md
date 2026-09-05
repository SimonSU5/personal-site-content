---
title: vxlan学习笔记
excerpt: vxlan学习笔记-分布式网关vxlan
category: 数通
tags:
  - 数通
cover: assets/covers/数通学习笔记-封面.png
date: 2026-09-03
published: true
---
## 分布式网关做法

分布式网关主要目的是防止形成发卡流量。
![[Pasted image 20260904145931.png]]
PC1和PC4通信，使用CE1为共同网关的情况下，流量会进入CE1之后，再从CE1流出，这部分是不必要的流量，最好能够直接CE2-R1-CE3。此时的设计思路是CE2和CE3这两个LEAF作为共同的网关。以下是一个例子：
PC1：192.168.1.1 PC4：192.168.2.4
CE2 vbdif 100：192.168.1.254 mac 0000-005e-0001
CE2 vbdif 200：192.168.2.254 mac 0000-005e-0002
CE3 vbdif 100：192.168.1.254 mac 0000-005e-0001
CE3 vbdif 200：192.168.2.254 mac 0000-005e-0002

CE2和CE3的同网段网关使用同一套mac地址和IP地址，如果这样设计，**CE2和CE3就在逻辑上能够被看作是一个设备**，流量也不会到CE1去转发。具体怎么实现呢？**类比普通的v4二层和三层网络，关键设计思路：**
	1. 同BD通信，通过查询**BD内二层表**。使用arp广播+vxlan隧道arp便可获取。
	2. 不同BD通信，必须通过三层网关VBDIF转发——两个不同的VBDIF必须要在同一台设备上才可以。
	3. 任播设计：**【关键】 一台CE设备，不可能知道另一台设备下的直连路由。想要把两台设备逻辑上看作一台设备，这两台设备要分别告知对方自己的直连路由下面有什么设备。**
		1. CE2将直连主机的32位地址作为目的地址，自己的vxlan隧道nve接口作为下一跳告诉CE1。——**本质为路由发布，即IRB路由**
			1. 若要实现这个目标，首先要选用bgp，因为只有bgp是应用层路由发布协议，和vxlan一样，无需理会中间的设备，只需路由可达即可。
			2. 这个路由发布并非普通ipv4-unicast，必须是evpn类型的路由发布
			3. vxlan二层的arp发布，使用evpn的rd+rt+mac+IP+二层vni来传递了。——这条arp会被放在**BD二层表**中。**顺便提一嘴，实际起到控制面作用的只有rt和rd。转发平面的时候按照二层vni来识别到底是哪个BD。**
			4. 根据上述条件，这条三层的evpn路由需要增加标识来告诉NVE设备本条路由是一个vxlan的三层路由，并且必须存放在一个单独的三层路由转发表内。——**这个转发表就是vpn实例的转发表。** 很明显这条路由需要被新加上一层标签，以此作为三层转发的标记，因此需要在vpn实例中配置一个新的rt，rd。
		2. 

| 转发过程                                   | 查表                        | 查看命令                                                                                                                                                                               | 关键关联                                                                                               |
| -------------------------------------- | ------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------- |
| 同leaf同bd转发<br>192.168.1.1-192.168.1.5  | BD内**二层表**                | dis mac-address bridge-domain 100<br>![[Pasted image 20260904150645.png]]                                                                                                          | vlan 10 <->int g 1/0/1.100 <-> bri 100<br>vlan 50 <->int g 1/0/1.500 <-> bri 100                   |
| 同leaf不同bd转发<br>192.168.1.1-192.168.2.2 | vbdif同设备跨网段转发，检查direct路由表 | dis arp vpn-instance l3-evpn/dis arp<br>![[Pasted image 20260904151631.png]]<br>dis ip routing-table vpn-instance l3-evpn protocol direct <br>![[Pasted image 20260904152738.png]] | vlan 10 <->int g 1/0/1.100 <-> bri 100（绑定rt rd rt evpn rd evpn） **<-> vbdif 100 <-> vpn instance** |
| 不同leaf同bd转发                            |                           | <br>![[Pasted image 20260904152909.png]]<br>dis mac-address bridge-domain 100<br>![[Pasted image 20260904170559.png]]                                                              |                                                                                                    |
| 不同leaf不同BD转发                           |                           |                                                                                                                                                                                    |                                                                                                    |
|                                        |                           |                                                                                                                                                                                    |                                                                                                    |
|                                        |                           |                                                                                                                                                                                    |                                                                                                    |

| 转发过程 | 建表过程 |     |
| ---- | ---- | --- |
|      |      |     |
|      |      |     |
|      |      |     |
|      |      |     |

![[Pasted image 20260904114158.png]]


---
## 分布式网关EVPN配置

**CE1：**
1. 作为bgp路由反射器。因为这个路由反射器中
没有bridge domain的存在，所以需要把rt过滤关掉，
防止把CE2发向CE3的有RT的报文过滤掉。配置是在bgp evpn
视图下，使用undo policy vpn-target
2. 作为irb的路由反射器，需要在bgp evpn下打开通告irb路由

**CE2：**
1. evpn-overlay enable

2. bgp 100
	 路由反射器IBGP邻居
	 peer 10.10.10.10 as-number 100
	 peer 10.10.10.10 connect-interface LoopBack0
	 #
	 ipv4-family unicast
	  peer 10.10.10.10 enable
	 #
	 使能l2vpn
	 l2vpn-family evpn
	  policy vpn-target
	  peer 10.10.10.10 enable
	  peer 10.10.10.10 advertise irb //通告irb路由

3. vpn实例
	ip vpn-instance l3-evpn
	 ipv4-family
	  route-distinguisher 100:200
	  vpn-target 100:200 export-extcommunity
	  vpn-target 100:200 export-extcommunity evpn
	  vpn-target 100:200 import-extcommunity
	  vpn-target 100:200 import-extcommunity evpn
	 vxlan vni 5010 // 三层vni

4. vxlan vni绑定BD
	bridge-domain 100
	 vxlan vni 100
	 evpn
	  route-distinguisher 100:10
	  vpn-target 100:100 export-extcommunity
	  vpn-target 100:200 export-extcommunity //必须写这条，生成交叉路由
	  vpn-target 100:100 import-extcommunity
	
	bridge-domain 200
	 vxlan vni 200
	 evpn
	  route-distinguisher 200:20
	  vpn-target 200:200 export-extcommunity
	  vpn-target 100:200 export-extcommunity //必须写这条，生成交叉路由
	  vpn-target 200:200 import-extcommunity

5. bd vni网关
	interface Vbdif100
	 ip binding vpn-instance l3-evpn
	 ip address 192.168.1.254 255.255.255.0
	 mac-address 0000-005e-0001 // 网关地址一致，必须要有一样的mac地址
	 vxlan anycast-gateway enable //任播启用
	 arp collect host enable
	#
	interface Vbdif200
	 ip binding vpn-instance l3-evpn
	 ip address 192.168.2.254 255.255.255.0
	 mac-address 0000-005e-0002
	 vxlan anycast-gateway enable
	 arp collect host enable

6. nve接口配置
	source 20.20.20.20
	vni 100 head-end peer-list protocol bgp // 模拟器有bug，这个先不要配置
	vni 200 head-end peer-list protocol bgp

7. 子接口绑定BD
	interface GE1/0/1.100 mode l2
	 encapsulation dot1q vid 10
	 bridge-domain 100
	interface GE1/0/1.200 mode l2
	 encapsulation dot1q vid 20
	 bridge-domain 200
	interface GE1/0/1.500 mode l2
	 encapsulation dot1q vid 50
	 bridge-domain 100

---
## 配置注意点

之前写的配置中有这样一条注释：
bridge-domain 100
 vxlan vni 100
 evpn
  route-distinguisher 100:10
  vpn-target 100:100 export-extcommunity
  vpn-target 100:200 export-extcommunity //必须写这条，生成交叉路由
  vpn-target 100:100 import-extcommunity
**vpn-target 100:200 export-extcommunity //必须写这条，生成交叉路由**
这条路由主要出现在以下图片的红框内。如果不配置这条rt，红框内的100：200不会出现，因此这条路由会被vpn instance拒收。因为vpn instance中已经配置了irt为100：200。回忆一下，这里和mpls vpn加rt的设计原则不太一样。最早接触到mpls vpn的时候，rt 和rd会被配置在ip vpn-instance中，接口绑定这个实例后，无需再配置rt就可以直接给ipv4地址加上rd和rt。**而在这里，[vbdif<-->BD]在绑定配置了rd和rt的vpn实例后，还需要再再BD中配置rt在报文中添加rt，所以vpn实例中的rt和rd可能更多的只起到过滤的作用。**
![[Pasted image 20260904115523.png]]

---

## 配置检查命令

1. dis bgp evpn all routing-table

2. dis bgp evpn all routing-table mac-route xxx

3. dis bgp evpn peer

4. dis mac-address bridge-domain xxx

5. dis ip routing-table vpn-instance xxx