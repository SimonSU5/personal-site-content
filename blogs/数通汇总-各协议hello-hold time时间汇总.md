---
title: 各协议hello-hold time时间汇总
excerpt: 各协议hello-hold time时间汇总
category: 学习笔记
tags:
  - 路由协议
  - 数通
  - 学习笔记
cover: assets/covers/数通学习笔记-封面.png
date: 2026-08-06
published: true
---
# 华为设备 全协议 Keep‑Alive / Hello / 定时器完整参数（仅华为 VRP 默认值，HCIE‑Datacom 专用）

> 名词释义
> 
> Hello‑interval：邻居保活报文发送周期
> 
> Hold‑time / 死亡超时：连续收不到 Hello 判定邻居 Down
> 
> 健壮值 Robustness‑Value (RV) 华为默认 = 2

## 一、生成树协议 STP/RSTP/MSTP/PVST+（华为全部版本默认）

表格

|定时器|默认数值|说明|
|---|---|---|
|Hello‑Time|2 s|根桥发送 BPDU 周期，取值范围 1～10s|
|Max‑Age|20 s|BPDU 报文 Message‑Age 到达 20‑s，视作失效|
|Forward‑Delay|15 s|侦听、学习状态停留时长|
|边缘端口 BPDU 超时时间|3×Hello‑Time = 6 s|收到 BPDU 之后关闭边缘端口的计时器|

## 二、三层组播协议（华为原厂默认）

### 1. IGMP‑v2

表格

|参数|华为默认|解释|
|---|---|---|
|普遍组查询间隔 Query‑interval|60 s|接口周期性发送全网查询报文|
|最大响应时间 Max‑Resp‑Time|10 s|主机随机 0~10s 回复成员报告|
|特定组查询间隔 Last‑Member‑Query‑Interval|1 s|收到 Leave 报文后发送查询的周期|
|最后成员查询发送次数|2|连发两次|
|组成员超时时间|130 s<br><br>计算公式：Query‑interval × RV + Max‑Resp‑Time<br><br>60*2+10=130s|接口侧 IGMP 成员表项老化时长|

### 2. IGMP‑Snooping 二层交换机

表格

|参数|华为默认值|
|---|---|
|VLAN 普遍组查询周期|60 s|
|最大响应时间|10 s|
|特定组查询间隔|1 s|
|路由器端口老化定时器|180 s|
|成员端口老化时间|130 s|

### 3. MLD（IPv6‑IGMP）

- 普遍查询间隔：60 s
- 最大响应时间：10 s
- 特定组查询间隔：1 s
- 组播成员超时：130 s

### 4. PIM‑DM / PIM‑SM

表格

|参数|华为默认|
|---|---|
|PIM Hello 发送间隔|30 s|
|PIM Hold‑Time（邻居死亡超时）|90 s（Hello × 3）|

## 三、IGP 路由协议

表格

|协议类型|Hello 间隔|Hold‑Time (死亡超时)|备注|
|---|---|---|---|
|OSPFv2 / OSPFv3 广播、P2P、P2MP|10 s|40 s（Hello×4）|NBMA 默认同样 10/40|
|IS‑IS（P2P、LAN）|10 s|30 s（Hello×3）|L1、L2 共用参数|
|EIGRP（华为兼容实现）广播链路|5 s|15 s|Hello*3|
|EIGRP NBMA 低速链路|60 s|180 s|Hello*3|

## 四、BGP、MPLS‑LDP

表格

|协议|Keepalive 报文周期|Hold‑Time|
|---|---|---|
|BGP|60 s|180 s|
|LDP|5 s|15 s（Hello ×3）|

> BGP 协商规则：两端取较小 Hold‑Time；keepalive = HoldTime ÷ 3；Hold‑Time=0 关闭邻居探测

## 五、二层可靠性协议（VRP 默认）

表格

|协议|Hello 间隔|故障判定超时|
|---|---|---|
|VRRP|1 s|3 s|
|LACP|慢周期 30 s；快周期 1 s|发送周期 × 3|
|RRPP|1 s|3 s|
|ERPS|5 s|15 s|

## 六、接入、隧道协议

表格

|协议|保活周期|超时时间|
|---|---|---|
|PPPoE LCP Echo‑Request|30 s|90 s（3 次没有应答断开）|

## 七、BFD 双向转发检测（华为默认参数）

- 最小发送间隔 min‑tx‑interval：100 ms
- 最小接收间隔 min‑rx‑interval：100 ms
- 检测倍数 detect‑multiplier：3
- 检测超时 = 100 × 3 = 300ms

## 八、各种表项老化定时器（华为设备）

表格

|条目类型|默认老化时长|
|---|---|
|ARP 表项|1200 s（20 min）|
|IPv6 ND 邻居缓存|1800 s（30 min）|
|DHCP‑Snooping 绑定表老化时间|300 s|

## 九、传输层 & 应用层保活

1. TCP（Linux 内核默认，华为设备 Linux 平台一致）

- tcp_keepalive_time=7200 s（空闲 2h 开启探测）
- tcp_keepalive_intvl=75 s
- tcp_keepalive_probes=9

2. SSH 服务端空闲保活：300 s
3. Nginx keepalive_timeout：60 s

|协议|Hello|Hold‑Time|
|---|---|---|
|OSPF|10s|40s|
|IS‑IS|10s|30s|
|BGP|60s|180s|
|LDP|5s|15s|
|PIM|30s|90s|
|VRRP|1s|3s|
|STP Hello|2s|Max‑Age 20s|
|IGMP 查询周期|60s|成员超时 130s|