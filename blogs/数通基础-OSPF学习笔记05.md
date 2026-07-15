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

## OSPF防环

1. 要求OSPF非骨干区域只能和骨干区域相连——杜绝多个区域成环
	1. 非骨干区域只能和骨干区域相连，形成**类似**星型组网（并不是真正的星型组网，因此还存在环路可能）
	2. 只有非骨干区域和骨干区域链接的路由器才能作为ABR，才能产生和转发3类LSA。这样3类LSA在非骨干区域相连的路由器之间开环。
2. 要求 ABR不往回发3类LSA
3. ABR的非骨干口收到3类LSA时，不能参与路由计算
	1. 看下方拓扑，当R3 area1接口收到3类LSA，不再参与路由计算，也就是不再转发回 area0。这样就不会导致路由环路
	2. 问题： 当area0出现**骨干不连续问题**也就是图中虚线断开时，R3收到LSA不转发回area0，因此r1和R2不通。但实际上可以从area1绕回。这种情况需要启用**虚链接**。

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

## 虚链接

如上所示，OSPF要求骨干区域必须是连续的。不连续会产生骨干区域不通问题。


```mermaid
graph LR
    subgraph Area0["Area 0"]
        direction LR
        R4((R4))
        R2((R2<br/>Router ID 10.0.2.2))
        R4 --- R2
    end
    
    subgraph Area1["Area 1"]
        direction LR
        R1((R1))
        R2v((R2))
        R3v((R3<br/>Router ID 10.0.3.3))
        R1 --- R2v
        R1 --- R3v
    end
    
    subgraph Area2["Area 2"]
        direction LR
        R3a((R3))
        R5((R5))
        R3a --- R5
    end
    
    R2v -.->|"虚连接 vlink"| R3v
    
    style R1 fill:#4472C4,stroke:#2F5597,color:#fff
    style R2 fill:#4472C4,stroke:#2F5597,color:#fff
    style R2v fill:#4472C4,stroke:#2F5597,color:#fff
    style R3 fill:#4472C4,stroke:#2F5597,color:#fff
    style R3v fill:#4472C4,stroke:#2F5597,color:#fff
    style R4 fill:#4472C4,stroke:#2F5597,color:#fff
    style R5 fill:#4472C4,stroke:#2F5597,color:#fff
    style Area0 fill:#E7F3FF,stroke:#4472C4,stroke-dasharray: 5 5
    style Area1 fill:#FFF2E7,stroke:#ED7D31,stroke-dasharray: 5 5
    style Area2 fill:#E7F9E7,stroke:#70AD47,stroke-dasharray: 5 5
```

---

### 拓扑要点

| 元素 | 说明 |
|------|------|
| **R2 (10.0.2.2)** | Area 0 的 ABR，通过 Area 1 与 R3 建立虚连接 |
| **R3 (10.0.3.3)** | Area 2 的 ABR，通过 Area 1 与 R2 建立虚连接 |
| **虚连接** | 逻辑隧道，穿越 Area 1，使 Area 2 能访问 Area 0 |
| **R1** | Area 1 内部路由器，作为虚连接的 Transit 区域 |
| **R4 / R5** | 各自区域的内部路由器 |

### 配置命令

```bash
# R2 配置
[R2-ospf-1] ospf 1
[R2-ospf-1] area 1
[R2-ospf-1-area-0.0.0.1] vlink-peer 10.0.3.3

# R3 配置
[R3-ospf-1] ospf 1
[R3-ospf-1] area 1
[R3-ospf-1-area-0.0.0.1] vlink-peer 10.0.2.2
```