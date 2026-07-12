---
title: RIP学习笔记
excerpt: RIP路由协议学习笔记
category: 数通
tags:
  - 数通
  - 路由协议
  - IGP
cover: assets/covers/RIP学习笔记-封面.png
date: 2026-07-06
published: false
---

## RIP协议拓扑图

### 基础网络拓扑

```mermaid
%%{init: {'theme': 'base', 'themeVariables': { 'primaryColor': '#fff', 'primaryTextColor': '#000', 'primaryBorderColor': '#333', 'lineColor': '#333', 'secondaryColor': '#e6f3ff', 'tertiaryColor': '#fff'}}}%%
graph TB
    subgraph Network["网络拓扑结构"]
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

    style PC1 fill:#90EE90,stroke:#333,stroke-width:2px
    style PC2 fill:#90EE90,stroke:#333,stroke-width:2px
    style PC3 fill:#90EE90,stroke:#333,stroke-width:2px
    style R1 fill:#87CEEB,stroke:#333,stroke-width:2px
    style R2 fill:#87CEEB,stroke:#333,stroke-width:2px
    style R3 fill:#87CEEB,stroke:#333,stroke-width:2px
```

### RIP路由信息交换

RIP基于**路由表通告**来自动泛洪路由信息。

```mermaid
%%{init: {'theme': 'base', 'themeVariables': { 'primaryColor': '#fff', 'primaryTextColor': '#000', 'primaryBorderColor': '#333', 'lineColor': '#333', 'secondaryColor': '#e6f3ff', 'tertiaryColor': '#fff'}}}%%
flowchart LR
    Start([开始]) --> Step1[第一步: 初始状态<br/>路由器启动，只有直连网络]
    Step1 --> Step2[第二步: RIP广播<br/>每30秒向邻居广播路由表<br/>使用UDP 520端口]
    Step2 --> Step3[第三步: 路由计算<br/>接收更新，计算跳数<br/>最大跳数为15]
    Step3 --> Step4[第四步: 收敛完成<br/>网络稳定，路由表完整]
    Step4 --> End([完成])

    style Start fill:#90EE90,stroke:#333,stroke-width:2px
    style End fill:#90EE90,stroke:#333,stroke-width:2px
    style Step1 fill:#FFE4B5,stroke:#333,stroke-width:2px
    style Step2 fill:#98FB98,stroke:#333,stroke-width:2px
    style Step3 fill:#87CEEB,stroke:#333,stroke-width:2px
    style Step4 fill:#DDA0DD,stroke:#333,stroke-width:2px
```

### 路由表变化详解

```mermaid
%%{init: {'theme': 'base', 'themeVariables': { 'primaryColor': '#fff', 'primaryTextColor': '#000', 'primaryBorderColor': '#333', 'lineColor': '#333', 'secondaryColor': '#e6f3ff', 'tertiaryColor': '#fff'}}}%%
graph TB
    subgraph Initial["第一步: 初始状态"]
        I_R1["Router1路由表:<br/>• 192.168.1.0/24 (直连, hop:0)<br/>• 172.16.1.0/24 (直连, hop:0)"]
        I_R2["Router2路由表:<br/>• 192.168.2.0/24 (直连, hop:0)<br/>• 172.16.1.0/24 (直连, hop:0)<br/>• 172.16.2.0/24 (直连, hop:0)"]
        I_R3["Router3路由表:<br/>• 192.168.3.0/24 (直连, hop:0)<br/>• 172.16.2.0/24 (直连, hop:0)"]
    end

    subgraph Update["第二步: RIP广播 (每30秒)"]
        U1["Router1 → Router2<br/>UDP:520<br/>包含: 192.168.1.0/24"]
        U2["Router2 → Router1<br/>UDP:520<br/>包含: 192.168.2.0/24, 172.16.2.0/24"]
        U3["Router2 → Router3<br/>UDP:520<br/>包含: 192.168.2.0/24, 172.16.1.0/24"]
    end

    subgraph Complete["第三步: 路由学习完成"]
        C_R1["Router1路由表:<br/>• 192.168.1.0/24 (直连, 0)<br/>• 172.16.1.0/24 (直连, 0)<br/>• 192.168.2.0/24 (R2, 1)<br/>• 172.16.2.0/24 (R2, 1)<br/>• 192.168.3.0/24 (R2, 2)"]
        C_R2["Router2路由表:<br/>• 192.168.2.0/24 (直连, 0)<br/>• 172.16.1.0/24 (直连, 0)<br/>• 172.16.2.0/24 (直连, 0)<br/>• 192.168.1.0/24 (R1, 1)<br/>• 192.168.3.0/24 (R3, 1)"]
        C_R3["Router3路由表:<br/>• 192.168.3.0/24 (直连, 0)<br/>• 172.16.2.0/24 (直连, 0)<br/>• 192.168.2.0/24 (R2, 1)<br/>• 172.16.1.0/24 (R2, 1)<br/>• 192.168.1.0/24 (R2, 2)"]
    end

    Initial --> Update
    Update --> Complete

    style I_R1 fill:#FFE4B5,stroke:#333
    style I_R2 fill:#FFE4B5,stroke:#333
    style I_R3 fill:#FFE4B5,stroke:#333
    style U1 fill:#98FB98,stroke:#333
    style U2 fill:#98FB98,stroke:#333
    style U3 fill:#98FB98,stroke:#333
    style C_R1 fill:#DDA0DD,stroke:#333
    style C_R2 fill:#DDA0DD,stroke:#333
    style C_R3 fill:#DDA0DD,stroke:#333
```

### RIP防环机制

```mermaid
%%{init: {'theme': 'base', 'themeVariables': { 'primaryColor': '#fff', 'primaryTextColor': '#000', 'primaryBorderColor': '#333', 'lineColor': '#333', 'secondaryColor': '#e6f3ff', 'tertiaryColor': '#fff'}}}%%
flowchart TB
    R1[Router1] -->|第一步: 水平分割<br/>不向入接口发送| R2[Router2]
    R2 -->|第二步: 毒性逆转<br/>标记hop不可达| R1
    R2 -->|第三步: 路由毒化<br/>标记hop为16| R3[Router3]
    R3 -->|第四步: 抑制计时器<br/>180秒内不信任更新| R2

    R1 -.->|直连网络失效| Net[Network<br/>10.0.0.0/24]
    R2 -.->|通告hop为16| Net
    R3 -.->|设置抑制计时器| Net

    style R1 fill:#FF6B6B,stroke:#333,stroke-width:2px
    style R2 fill:#FF6B6B,stroke:#333,stroke-width:2px
    style R3 fill:#FF6B6B,stroke:#333,stroke-width:2px
    style Net fill:#FFD93D,stroke:#333,stroke-width:2px
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

### RIP缺陷
1. 开销计算方式：一条百兆线路，一条千兆路径。若百兆跳数少，则会选择百兆路径。这样选出来的是瓷釉路径。
2. 开销上线和收敛方式的缺陷：收敛方式通过递归收敛，速度慢因此有上限（16跳），不适合网络较大场景。