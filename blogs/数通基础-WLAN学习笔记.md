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

# 华为 AC‑AP 基础上线配置

> 适用：AC 控制器管理瘦 AP，CAPWAP 隧道 + WLAN 业务部署

## 一、AP 上线流程

### 1. AP 通过 DHCP 获取 IP 地址

1. 创建 AP 管理 VLAN
2. 在 AC 设备配置 DHCP 地址池，为瘦 AP 分配管理 IP

```
# 1、配置CAPWAP隧道源接口
capwap source interface Vlanif 100

# 2、创建AP分组
ap-group name hcia

# 3、MAC地址认证注册AP
ap-id 1 ap-mac 00e0-fc26-7030
ap-name ap1
ap-group hcia
```

## 二、无线业务参数配置

### 1. SSID 模板

```
ssid-profile name wlan-ssid
ssid huawei
```

### 2. 安全模板（WPA‑WPA2‑PSK+AES 加密）

```
security-profile name wlan-sec
security wpa-wpa2 psk pass-phrase huawei@123 aes
```

### 3. VAP 业务模板（绑定业务 VLAN、SSID、安全策略）


```
vap-profile name wlan-vap
 service-vlan vlan-id 10
 ssid-profile wlan-ssid
 security-profile wlan-sec
```

### 4. 将 VAP 模板下发至 AP 组射频

```
ap-group name huawei
vap-profile wlan-vap wlan 1 radio all
```

## 三、配置思路梳理

| 阶段      | 功能说明                                                        |
| ------- | ----------------------------------------------------------- |
| AP 管理   | Vlanif100 为 CAPWAP 源接口、DHCP 分配 AP 管理 IP、MAC 认证上线 AP、AP 分组管理 |
| WLAN 模板 | SSID 名称、WiFi 加密策略、业务 VLAN、模板下发到 AP 射频                       |
	