---
title: SDWAN学习笔记
excerpt: SDWAN学习笔记
category: 数通
tags:
  - 数通
  - 路由协议
  - SDWAN
cover: assets/covers/数通学习笔记-封面.png
date: 2026-08-25
published: true
---
# 智能选路

1. SAC（smart application control）
	1. 简介
		1. 以应用为维度管理网络
		2. 帮助路由交换设备识别分类应用
		3. 通过SA（业务感知）和FPI（首包识别）
	2. 总思想
		1. 从SAC特征库进行SAC检测
		2. 匹配到某种应用，就对这种应用使用某种策略（QoS，流量策略等）
	3. 流程
		1. 业务流量首先根据五元组查看识别记录，有就直接根据策略和转发表处理
		2. 没有识别记录，通过FPI守包识别流量，如果能够识别出流量种类，再根据策略和转发表处理
		3. 如果FPI没有识别出，则通过SA识别业务。识别出流量种类，再根据策略和转发表处理
	4. 特征
		1. SA需要多个包进行分析
2. SPR（smart policy routing）
	1. 通过检测链路质量，来选择满足QoS的转发链路
	2. NQA业务选路
		1. 可以通过时延（D），抖动（Jitter），丢包率（L），cmi-method=D+J+L
		2. CMI=9000-cmi-method，越大链路质量越优

# SDWAN

## 主要功能

1. 解决混合链路（MPLS + 宽带 + 5G）下**动态智能选路**（PBR 最大短板）
2. 全自动 Overlay 全网隧道组网，不用手动 VPN
3. 本身支持策略，可适配多种转发策略
4. 集中管理（配置由NCE直接下发），链路质量全网可视

## SDWAN解决方案

| 角色  | 功能                                             | 产品形态        |
| --- | ---------------------------------------------- | ----------- |
| 管理层 | 1）网络业务编排2）网络性能监控与可视化3）网络运维4）网络设备的管理            | iMaster NCE |
| 控制层 | 1）路由和隧道信息分发2）IPsec 密钥交换3）VPN 拓扑定义4）NAT Stun 服务 | RR（AR 路由器）  |
| 网络层 | 1）EDGE：企业分支、总部、DC 以及云等站点的出口 CPE 设备2）GW：多租户网关设备 | AR 路由器      |

1. NCE 控制器

|层级|模块|具体内容|
|---|---|---|
|上层对接|增值业务|OSS/BSS、分析系统、其他应用|
|iMaster NCE 核心层|北向接口|RESTful、SNMP Trap、Syslog|
|iMaster NCE 核心层|业务功能|即插即用、流量策略、安全策略、可视化运维|
|iMaster NCE 核心层|基础功能|集群管理、多租户管理、隧道管理、设备配置、设备升级、日志管理、告警管理、网络巡检|
|iMaster NCE 核心层|南向接口|NETCONF、HTTP2.0|
|底层对接|网络设备|CPE、vCPE|
2. 网路层
	1. Edge设备——IP overlay隧道技术（GRE over ipsec）
	2. RR——路由反射器
		1. BGP路由传递
		2. 控制路由和网络拓扑
		3. RR和RR之间，RR和Edge之间建立BGP EVPN控制通道。RR控制Edge的路由收发等信息。
			1. TNP信息：主要建立overlay隧道，传递业务
		4. RR和Edge都通过管理通道收到NCE管理，管理通道使用NETCONF。
	3. GW——SDWAN网关
		1. 兼具传统MPLS等网络和SDWAN的功能，让sdwan和传统网络连接。
	4. 