---
title: RIP学习笔记
excerpt: RIP路由协议学习笔记
category: 数通
tags:
  - 数通
cover: assets/covers/image.jpg
date: 2026-07-06
published: false
---

## RIP协议拓扑图

### 基础网络拓扑

```mermaid
graph TB
    subgraph Network["网络拓扑"]
        PC1(("PC1<br/>192.168.1.10"))
        R1["Router1<br/>192.168.1.1"]
        PC2(("PC2<br/>192.168.2.10"))
        R2["Router2<br/>192.168.2.1"]
        PC3(("PC3<br/>192.168.3.10"))
        R3["Router3<br/>192.168.3.1"]
    end

    PC1 ---|"直连<br/>hop:0"| R1
    PC2 ---|"直连<br/>hop:0"| R2
    PC3 ---|"直连<br/>hop:0"| R3

    R1 ===|"172.16.1.0/24<br/>hop:1"| R2
    R2 ===|"172.16.2.0/24<br/>hop:1"| R3

    style PC1 fill:#90EE90
    style PC2 fill:#90EE90
    style PC3 fill:#90EE90
    style R1 fill:#87CEEB
    style R2 fill:#87CEEB
    style R3 fill:#87CEEB
```

### RIP路由信息交换

```mermaid
graph TB
    subgraph RIP_Update["RIP路由更新流程"]
        direction LR

        subgraph Phase1["阶段1: 初始状态"]
            R1_Table["Router1路由表:<br/>• 192.168.1.0/24 (直连, hop:0)<br/>• 172.16.1.0/24 (直连, hop:0)"]
            R2_Table["Router2路由表:<br/>• 192.168.2.0/24 (直连, hop:0)<br/>• 172.16.1.0/24 (直连, hop:0)<br/>• 172.16.2.0/24 (直连, hop:0)"]
            R3_Table["Router3路由表:<br/>• 192.168.3.0/24 (直连, hop:0)<br/>• 172.16.2.0/24 (直连, hop:0)"]
        end

        subgraph Phase2["阶段2: RIP广播 (每30秒)"]
            Update1["Router1 → Router2<br/>UDP:520<br/>包含: 192.168.1.0/24"]
            Update2["Router2 → Router1<br/>UDP:520<br/>包含: 192.168.2.0/24, 172.16.2.0/24"]
            Update3["Router2 → Router3<br/>UDP:520<br/>包含: 192.168.2.0/24, 172.16.1.0/24"]
        end

        subgraph Phase3["阶段3: 路由学习完成"]
            Final_R1["Router1路由表:<br/>• 192.168.1.0/24 (直连, 0)<br/>• 172.16.1.0/24 (直连, 0)<br/>• 192.168.2.0/24 (R2, 1)<br/>• 172.16.2.0/24 (R2, 1)<br/>• 192.168.3.0/24 (R2, 2)"]
            Final_R2["Router2路由表:<br/>• 192.168.2.0/24 (直连, 0)<br/>• 172.16.1.0/24 (直连, 0)<br/>• 172.16.2.0/24 (直连, 0)<br/>• 192.168.1.0/24 (R1, 1)<br/>• 192.168.3.0/24 (R3, 1)"]
            Final_R3["Router3路由表:<br/>• 192.168.3.0/24 (直连, 0)<br/>• 172.16.2.0/24 (直连, 0)<br/>• 192.168.2.0/24 (R2, 1)<br/>• 172.16.1.0/24 (R2, 1)<br/>• 192.168.1.0/24 (R2, 2)"]
        end
    end

    Phase1 --> Phase2
    Phase2 --> Phase3

    style R1_Table fill:#FFE4B5
    style R2_Table fill:#FFE4B5
    style R3_Table fill:#FFE4B5
    style Update1 fill:#98FB98
    style Update2 fill:#98FB98
    style Update3 fill:#98FB98
    style Final_R1 fill:#DDA0DD
    style Final_R2 fill:#DDA0DD
    style Final_R3 fill:#DDA0DD
```

### RIP防环机制

```mermaid
graph TB
    subgraph Loop_Prevention["RIP环路预防机制"]
        R1["Router1"]
        R2["Router2"]
        R3["Router3"]
        Net["Network<br/>10.0.0.0/24"]

        R1 -->|"水平分割<br/>不向入接口发送"| R2
        R2 -->|"毒性逆转<br/>标记hop=16(不可达)"| R1
        R2 -->|"路由毒化<br/>标记hop=16"| R3
        R3 -->|"抑制计时器<br/>180s内不信任更新"| R2

        R1 -.->|"直连网络失效"| Net
        R2 -.->|"R1通告hop=16"| Net
        R3 -.->|"学习到的路由<br/>设置抑制计时器"| Net
    end

    style R1 fill:#FF6B6B
    style R2 fill:#FF6B6B
    style R3 fill:#FF6B6B
    style Net fill:#FFD93D
```

## RIP协议核心概念

### 工作原理

1. **路由表初始化**: 路由器启动时，只包含直连网络
2. **路由广播**: 每30秒向邻居广播完整路由表
3. **路由计算**: 使用跳数作为度量，最大15跳，16跳表示不可达
4. **收敛时间**: 网络稳定时间较长，不适合大型网络

### 关键特性

| 特性 | 说明 |
|------|------|
| **度量** | 跳数(Hop Count)，最大15 |
| **更新周期** | 30秒 |
| **端口** | UDP 520 |
| **版本** | RIPv1(有类), RIPv2(无类, 支持VLSM) |
| **防环机制** | 水平分割、毒性逆转、路由毒化、抑制计时器 |

### RIP v1 vs RIPv2

| 特性 | RIPv1 | RIPv2 |
|------|-------|-------|
| 地址类型 | 有类路由 | 无类路由(VLSM) |
| 认证 | 不支持 | 明文/MD5认证 |
| 更新方式 | 广播(255.255.255.255) | 组播(224.0.0.9) |
| 路由标记 | 不支持 | 支持 |
| 下一跳 | 不支持 | 支持 |

