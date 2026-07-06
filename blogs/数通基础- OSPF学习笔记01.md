---
title: OSPF学习笔记01
excerpt: OSPF学习笔记
category: 数通
tags:
  - 数通
  - 路由协议
cover: assets/covers/image.jpg
date: 2026-07-06
published: false
---

## 链路状态路由协议——LSA泛洪

OSPF通过链路状态（拓扑信息+路由信息）通告路由情况。
运行OSPF的路由器会先建立邻居关系，之后彼此开始交互LSA（链路状态通告）
收到的LSA将放入LSDB中。