---
title: OSPF学习笔记05——防环
excerpt: OSPF学习笔记05——防环
category: 学习笔记
tags:
  - 路由协议
  - 数通
  - IGP
  - 学习笔记
cover: assets/covers/OSPF学习笔记-封面.png
date: 2026-07-15
published: false
---
# 区域间防环机制

1. 要求OSPF非骨干区域只能和骨干区域相连——杜绝多个区域成环
	1. 非骨干区域只能和骨干区域相连，形成**类似**星型组网（并不是真正的星型组网，因此还存在环路可能）
	2. 只有非骨干区域和骨干区域链接的路由器才能作为ABR，才能产生和转发3类LSA。这样3类LSA在非骨干区域相连的路由器之间开环。
2. 要求 ABR不往回发3类LSA
3. ABR的非骨干口收到3类LSA时，不能参与路由计算

```mermaid
graph TB
    subgraph "Area 0"
        R1((R1<br/>Router ID))
        R2((R2<br/>Router ID<br/>Loopback0: 10.0.2.2/32))
        R3((R3<br/>Router ID<br/>ABR))
        R4((R4<br/>Router ID<br/>ABR))
        
        R1 -.×.-|"P2P 断开"| R2
        R1 ---|"P2P"| R3
        R3 -.×.-|"P2P 断开"| R4
        R2 ---|"P2P"| R4
    end
    
    subgraph "Area 1"
        R5((R5<br/>Router ID))
        R6((R6<br/>Router ID))
        
        R3 ---|"P2P"| R5
        R4 ---|"P2P"| R6
        R5 ---|"P2P"| R6
    end
    
    style R1 fill:#4472C4,color:#fff
    style R2 fill:#4472C4,color:#fff
    style R3 fill:#70AD47,color:#fff
    style R4 fill:#70AD47,color:#fff
    style R5 fill:#4472C4,color:#fff
    style R6 fill:#4472C4,color:#fff
```