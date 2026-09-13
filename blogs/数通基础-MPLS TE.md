---
title: MPLS TE学习笔记
excerpt: MPLS TE学习笔记
category: 学习笔记
tags:
  - 路由协议
  - 数通
  - 学习笔记
  - 
cover: assets/covers/数通学习笔记-封面.png
date: 2026-08-24
published: true
---
# 广域网技术

1. MP-BGP
	1. 控制平面： MP-BGP通过不同地址族区分——L3VPNv6/L3VPNv6/L2VPN（vxlan），EVPN（GRE vxlan）
	2. 数据平面：MPLS/IP，SR-MPLS，SRv6
2. MP-BGP可以向后兼容BGP-4
3. MP-BGP update报文
	1. path attributes：扩展community RT
	2. MP_REACH_NLRI：rd+路由+下一跳
	3. MP_UNREACH_NLRI：撤销rd+路由+下一跳
4. 