---
title: OSPF学习笔记06——外部路由计算
excerpt: OSPF学习笔记06——外部路由计算
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
## 外部路由使用场景

```mermaid
flowchart TB
    subgraph Internet
        CLOUD["Internet 公网云"]
    end
    subgraph FW集群
        FW1["防火墙FW1"]
        FW2["防火墙FW2"]
    end
    subgraph OSPF_Area0 ["OSPF Area 0 骨干区域"]
        R0_1["骨干路由器R0-1"]
        R0_2["骨干路由器R0-2"]
    end
    subgraph Area1 ["OSPF Area 1"]
        R1_1["R1-1"]
        R1_2["R1-2"]
        PC1["终端用户组1"]
        PC2["终端用户组2"]
    end
    subgraph AreaN ["OSPF Area N"]
        RN_1["RN-1"]
        RN_2["RN-2"]
        PCN1["终端用户组N1"]
        PCN2["终端用户组N2"]
    end
    SERVER["通用服务器<br/>12.1.1.0/24"]

    %% 公网-防火墙连接
    CLOUD --- FW1
    CLOUD --- FW2
    %% 防火墙到骨干路由
    FW1 <--使用静态路由--> R0_1
    FW2 <--使用静态路由--> R0_2
    %% 防火墙之间互联 标注①
    FW1 -- 链路① OSPF已开启 --> FW2
    %% Area0内部互联
    R0_1 --- R0_2
    %% Area0 下联各区域ABR
    R0_1 --- R1_1
    R0_1 --- RN_1
    R0_2 --- R1_2
    R0_2 --- RN_2
    %% 区域内路由下联终端
    R1_1 --- PC1
    R1_2 --- PC2
    RN_1 --- PCN1
    RN_2 --- PCN2

    %% 关键链路②：R0-2 到服务器，未运行OSPF
    R0_2 -- 链路② <b>未开启OSPF</b> --> SERVER

    %% 美化样式
    classDef area0 fill:#e6f0ff,stroke:#0044bb,stroke-width:2px
    classDef areaX fill:#f0f5ff,stroke:#4466cc,stroke-width:1px
    classDef fwstyle fill:#ffeeee,stroke:#bb2222
    classDef serverstyle fill:#eef8ff,stroke:#006699
    class OSPF_Area0 area0
    class Area1,AreaN areaX
    class FW1,FW2 fwstyle
    class SERVER serverstyle
```

1. 情况1： 注意上图中未开启OSPF的骨干路由器，未开启OSPF时，这个通用服务器的路由网段不会宣告到下面的拓扑中。
2. 情况2： 注意上图中骨干路由器和防火墙集群时通过静态路由链接的，防火墙上方的所有网段之后静态路由配置到骨干路由器中。这样也不会通过OSPF协议通告到下方的路由器中。

## 外部路由引入相关概念

1. 路由器类型：ASBR 自治系统边界路由器
2. LSA类型：5类LSA-ASE LSA（AS external），AS外部路由，将域外路由在OSPF网络所有区域（除了[[数通基础-OSPF学习笔记07#stub区域]]和[[数通基础-OSPF学习笔记07#NSSA区域]]）内泛洪。

## 5类LSA——ASE LSA外部LSA报文组成

ASBR生成，用于描述AS外部的路由。——泛洪到整个

1. LSA头部
	1. link state type: 5 ASE
	2. link state id: 外部路由的目的网络地址
	3. advertise router id: 宣告LSA的router id
	4. E：0表示metric-type-1，1表示metric-type-2
	5. metric：到目的网络的度量
	6. network mask：外部路由的目的网络掩码

## 4类LSA——ASBR Summery LSA，ASBR

 ABR生成，用于描述ASBR的路由信息
 
 1. LSA头部
	1. link state type: 4 ASBR summery
	2. link state id: ASBR router id
	3. advertise router id: 宣告LSA的router id（ABR router id）
	4. metric：到目的网络的度量
	5. network mask：外部路由的目的网络掩码