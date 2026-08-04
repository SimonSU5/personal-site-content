---
title: WLAN学习笔记
excerpt: WLAN学习笔记
category: 学习笔记
tags:
  - 数通
  - 学习笔记
cover: assets/covers/数通学习笔记-封面.png
date: 2026-08-02
published: true
---
1. WLAN组网架构
	1. 胖AP架构：每台设备独立配置
	2. 瘦AP架构：使用AC+AP，CAPWAP控制协议通信
2. AC+AP组网
	1. 二层组网：AC和AP在同一vlan，小型网络
	2. 三层组网：可以承载几十台-上百台AP
3. AC+AP连接方式
	1. 直连式组网
	2. 旁挂式组网
4. BSS/SSID/BSSID
	1. BSS
		1. 一个AP所覆盖的范围
		2. 在一个BSS范围内，STA可以相互通信
	2. BSSID
		1. WiFi名称，是MAC地址
	3. SSID
		1. WiFi别名，日常WiFi名称
5. VAP 虚拟接入点
	1. 一个AP虚拟出多个AP
	2. 每个虚拟出的AP就是一个VAP。
6. ESS 扩展服务集
	1. 多个BSS但是同一个SSID组成的服务集合区域
7. WLAN工作流程
	1. AP获取IP地址上线，与AC建立链接
	2. WLAN业务下发
	3. STA接入：STA搜索到SSID并连接上线，接入网络
	4. WLAN业务数据转发