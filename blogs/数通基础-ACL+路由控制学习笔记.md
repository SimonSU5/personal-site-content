---
title: ACL+路由控制学习笔记
excerpt: ACL+路由控制学习笔记
category: 学习笔记
tags:
  - 数通
  - 学习笔记
cover: assets/covers/数通学习笔记-封面.png
date: 2026-08-02
published: true
---
## ACL访问控制列表

1. ACL组成
	1. acl number xxxx （2000-2999 basic，3000-3999 advanced，4000-4999 二层）
	2. rule 5（默认5步进）
	3. permit/deny
	4. 规则
		1. 基本ACL
			1. 源地址
			2. 分片信息
			3. 生效时间
		2. 高级ACL
			1. 源目IP
			2. 源目端口
			3. IP协议类型
			4. ICMP类型
			5. 生效时间段
		3. 二层ACL
			1. 源目MAC地址
			2. 二层协议
		4. 用户自定义ACL（5000-5999）
		5. 用户ACL（6000-6999）
	5. 默认在最尾加上全部deny
2. ACL匹配位置
	1. inbound
	2. outbound

---

### 路由控制

1. ACL
	1. config模式匹配：规则越小，越先匹配
2. IP前缀列表
	1. 优点：可以匹配路由的子网掩码
	2. 缺点：只能匹配路由信息，无法匹配流量
3. 结构
	1. ip ip-prefix name index xx permit/deny ip netmask greater-equal xx less-equal xx
	2. 前面的netmask规定了严格匹配
	3. 后面的掩码规定了子网长度
	4. 缺省在最后全部拒绝