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
	1. 看下方拓扑，当R3 area1接口收到3类LSA，不再参与路由计算，也就是不再转发回 area0。这样

```mermaid
graph TB
    subgraph Area0["Area 0"]
        direction TB
        R1((R1))
        R2((R2<br/>Loopback0<br/>10.0.2.2/32))
        R3((R3))
        R4((R4))
        
        R1 -.-x R2
        R1 --- R3
        R3 -.-x R4
        R2 --- R4
    end
    
    subgraph Area1["Area 1"]
        direction TB
        R5((R5))
        R6((R6))
        
        R3 --- R5
        R4 --- R6
        R5 --- R6
    end
    
    style R1 fill:#4472C4,stroke:#2F5597,color:#fff
    style R2 fill:#4472C4,stroke:#2F5597,color:#fff
    style R3 fill:#70AD47,stroke:#548235,color:#fff
    style R4 fill:#70AD47,stroke:#548235,color:#fff
    style R5 fill:#4472C4,stroke:#2F5597,color:#fff
    style R6 fill:#4472C4,stroke:#2F5597,color:#fff
    style Area0 fill:#E7F3FF,stroke:#4472C4
    style Area1 fill:#E7F9E7,stroke:#70AD47
```

4. 

