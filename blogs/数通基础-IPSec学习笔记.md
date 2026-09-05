---
title: IPSec学习笔记
excerpt: IPSec学习笔记
category: 数通
tags:
  - 数通
cover: assets/covers/数通学习笔记-封面.png
date: 2026-08-24
published: true
---
> 🔗 配套 eNSP 实验：[学习实验-ipsec配置](https://github.com/SimonSU5/network-lab/tree/main/学习实验-ipsec配置)

# IPsec v1（IKEv1）与 IKEv2 协商前置过程

> IPsec 本身是加密框架，**真正负责协商密钥、SA 的是 IKE 协议**：IKEv1 对应 IPsec v1，IKEv2 对应 IPsec v2。 IKE 本质是在协商出安全通道，再用这个通道去协商 IPsec 业务 SA。

## IKEv1（IPsec v1）分 2 个阶段

### 第一阶段（Phase1）：协商**IKE SA**（管理 SA）

> 这就是你说的 “前面在协商” 的部分，目的：建立一条加密安全的控制通道，保护后面第二阶段报文。 协商内容：

1. 协商 **IKE 策略**：加密算法、认证算法、DH 密钥交换组、生存时间
2. 执行 **DH 交换**：双方交换 DH 公钥，生成共享密钥
3. 交换身份、做身份认证（预共享密钥 / 证书）
4. 生成 IKE SA（管理 SA，双向），**后续 Phase2 所有报文全部被这个 IKE SA 加密保护**

IKEv1 第一阶段两种模式：

- **主模式 Main Mode**：6 条报文，身份加密，推荐
- **野蛮模式 Aggressive Mode**：3 条报文，身份明文，适合 NAT 场景

> Phase1 产出：**IKE SA（管理 SA）**，不处理业务流量，只用来做后续协商。

### 第二阶段（Phase2）：协商 **IPsec SA（业务 SA）**

跑在 Phase1 建立的加密通道内部。 协商内容：感兴趣流、IPsec 加密套件、PFS、SA 生存时间。 产出：**单向的 IPsec SA（in/out 各一条）**，真正用来加密用户业务 IP 报文。

---

## IKEv2（IPsec v2）：没有严格的 Phase1/Phase2 概念，换成 INIT、AUTH 交换

IKEv2 把流程重新封装，**前 2 条报文就是前置协商（IKE_SA_INIT）**，等价于 IKEv1 Phase1 的前半部分。

1. **IKE_SA_INIT（第 1、2 报文，前置协商）** 等价 IKEv1 Phase1 的算法协商 + DH 交换：

- 协商 IKE 加密、完整性算法、DH 组
- 交换 DH 公钥、随机 nonce
- **不做身份认证！身份认证放到下一组报文**

> 这一步完成，双方已经算出共享密钥，但还不知道对方是谁。

2. **IKE_AUTH（第 3、4 报文）** 基于上面算出的密钥做加密，交换身份、认证，**创建 IKE SA**。 紧接着立刻创建 CHILD_SA（等价 IKEv1 的 IPsec SA，业务 SA）。

> IKEv2 没有野蛮模式；NAT 检测、DPD 原生内置。

|项目|IKEv1(IPsec v1)|IKEv2(IPsec v2)|
|---|---|---|
|前置协商|Phase1（主 / 野蛮模式），完整做完算法 + DH + 身份认证，生成 IKE SA|IKE_SA_INIT：仅算法 + DH，**不认证身份**；身份认证延后到 IKE_AUTH|
|阶段划分|严格 Phase1 (管理) → Phase2 (业务)|IKE_SA (IKE SA，管理) → CHILD_SA (业务 SA)|
|报文开销|主模式 6 报文，野蛮 3 报文|正常 4 报文完成 IKE + 第一条业务 SA|


---

# 广域网互联NAT穿越技术

## NAT专有术语

|Tuple|组成字段|应用场景|
|---|---|---|
|二元组|源 IP，目的 IP|简单 NAT，不关注端口|
|三元组|源 IP，源端口，目的 IP|**Easy‑IP/NAPT，最常用**|
|五元组|源 IP，源端口，目的 IP，目的端口，协议号|NAT‑Server、策略 NAT，区分同 IP 不同业务|

## NAT类型

1. 现网中的NAPT和easy-ip基本都是对称NAT
2. 如果有NAT穿越的需求，必须要用Cone型的NAT

| NAT 类型    | 英文名称                     | 映射规则                                           | 入站过滤规则                    | P2P 打洞能力 |
| --------- | ------------------------ | ---------------------------------------------- | ------------------------- | -------- |
| 全锥 NAT    | Full Cone NAT            | 内网 (私 IP: 私端口)→任意外网，**固定公 IP: 公端口**            | 任意外网 IP: 端口均可访问该映射        | 强        |
| 地址受限锥 NAT | Restricted Cone NAT      | 内网 (私 IP: 私端口)→任意外网，**固定公 IP: 公端口**            | 仅放行内网主动访问过的**源 IP**，端口不限  | 中        |
| 端口受限锥 NAT | Port Restricted Cone NAT | 内网 (私 IP: 私端口)→任意外网，**固定公 IP: 公端口**            | 仅放行内网主动访问过的**源 IP + 源端口** | 弱        |
| 对称 NAT    | Symmetric NAT            | 内网 (私 IP: 私端口) 访问**不同目的 IP: 端口，分配不同公 IP: 公端口** | 仅响应曾经访问过的**目的 IP + 目的端口** | 几乎无法打洞   |

## NAT穿越

1. ALG
	1. 在多端口协议的情况下，比如SIP FTP等，一般控制信息的端口固定，但是转发端口是应用层随机指定，无法用目的NAT进行转化。
	2. 查看报文字段应用层的载荷信息，解析需要开放的端口。
	3. 缺陷：
		1. 对于新协议需要升级ALG内容。
		2. 不支持私有的协议
		3. 一般设备不支持
		4. 一般作为多端口协议的服务器场景下的问题。
2. STUN
	1. 见sdwan

---

# IPsec/IKE 全套术语缩写对照表

> 范围：IKEv1、IKEv2、ESP/AH、密钥派生、SA 相关，结合前面 DH、PRF、SPI 知识点。

表格


| 缩写                  | 完整英文                                                      | 中文释义                                                    |               |
| ------------------- | --------------------------------------------------------- | ------------------------------------------------------- | ------------- |
| **IPsec**           | Internet Protocol Security                                | 互联网协议安全，IP 层安全框架                                        |               |
| **IKE**             | Internet Key Exchange                                     | 互联网密钥交换，用来协商 SA 的控制协议                                   | 规定了密钥交换的流程    |
| **ISAKMP**          | Internet Security Association and Key Management Protocol | 互联网安全关联密钥管理协议，IKEv1 底层报文格式框架                            | 规定了密钥交换时的报文结构 |
| **SA**              | Security Association                                      | 安全关联，单向的策略 + 密钥集合，IPsec 最核心实体                           |               |
| **SPI**             | Security Parameters Index                                 | 安全参数索引，32bit，SA 的索引 ID，三元组定位 SA                         |               |
| **ESP**             | Encapsulating Security Payload                            | 封装安全载荷；提供加密 + 完整性 + 防重放                                 |               |
| **AH**              | Authentication Header                                     | 认证头；只做完整性校验，**不加密**，现在几乎不用                              |               |
| **Mode‑transport**  | Transport Mode                                            | 传输模式：保护上层报文，IP 头不变，端到端场景                                |               |
| **Mode‑tunnel**     | Tunnel Mode                                               | 隧道模式：完整封装原始 IP 报文，新增外层 IP 头，站点‑站点 VPN                   |               |
| **DH**              | Diffie‑Hellman                                            | 迪菲‑赫尔曼密钥交换；不安全网络协商得到原始共享秘密原料                            |               |
| **PRF**             | Pseudo‑Random Function                                    | 伪随机函数；把 DH 原始秘密、nonce、PSK 衍生出各类工作密钥                     |               |
| **PFS**             | Perfect Forward Secrecy                                   | 完美前向保密；每次业务 SA 重新执行一轮 DH，旧密钥泄露不影响历史流量                   |               |
| **PSK**             | Pre‑Shared Key                                            | 预共享密钥，简单认证方式，设备间预先配置相同密码                                |               |
| **Ni / Nr**         | Nonce‑Initiator / Nonce‑Responder                         | 发起方 / 响应方随机数 nonce，防重放攻击，密钥推导输入材料                       |               |
| **Ci / Cr**         | Cookie‑Initiator / Cookie‑Responder                       | IKEv1 的 Cookie，IKE SA 的标识符，抗 DoS，替代 SPI 给 IKE 管理 SA 使用  |               |
| **KE**              | Key Exchange payload                                      | 密钥交换载荷；携带 DH 公钥，IKE 报文中交换 DH 公钥                         |               |
| **ID**              | Identification payload                                    | 身份载荷；IKE 身份 ID，匹配对等体策略、参与认证签名计算                         |               |
| **SKEYID**          | Secret Key ID                                             | IKE 根密钥种子，PSK 模式由 PSK+Ni                                | Nr 经 PRF 算出   |
| **SKEYID_d**        | SKEYID‑derived                                            | 衍生密钥，专门用来给 Phase2/CHILD_SA 生成业务 SA 密钥的种子                |               |
| **SKEYID_a**        | SKEYID‑authentication                                     | IKE SA 完整性 / 认证密钥，校验 IKE 报文完整性                          |               |
| **SKEYID_e**        | SKEYID‑encryption                                         | IKE SA 加密密钥，加密 IKE 协商报文（Phase2 报文）                      |               |
| **HASH_I / HASH_R** | Hash‑Initiator / Hash‑Responder                           | 发起方 / 响应方认证哈希值，PSK 认证，校验对端身份合法性                         |               |
| **SA payload**      | Security Association payload                              | SA 载荷；协商加密、认证、DH 套件的载荷                                  |               |
| **TS**              | Traffic Selector                                          | 流量选择器 (IKEv2)，等价 IKEv1 Phase2 感兴趣流，匹配需要保护的网段            |               |
| **NAT‑D**           | NAT‑Detection                                             | NAT 探测载荷；检测链路中间是否存在 NAT 设备                              |               |
| **NATT**            | NAT‑Traversal                                             | NAT 穿越；ESP 封装 UDP 4500，解决中间 NAT 改写 IP 导致 ESP 校验失败问题     |               |
| **UDP‑4500**        | UDP port 4500                                             | NAT‑Traversal 端口；NAT 存在时 IKE/ESP 走 UDP4500              |               |
| **UDP‑500**         | UDP port 500                                              | 标准 IKE 协商端口，无 NAT 场景使用                                  |               |
| **SEQ**             | Sequence Number                                           | ESP 序列号，4 字节，防重放攻击，单向递增                                 |               |
| **IV**              | Initialization Vector                                     | 初始化向量，加密算法的初始随机向量                                       |               |
| **ICV**             | Integrity Check Value                                     | 完整性校验值，ESP/AH 尾部校验哈希结果，防篡改                              |               |
| **DPD**             | Dead Peer Detection                                       | 死亡对等体检测；周期性探测对端是否存活，清理失效 SA                             |               |
| **Phase‑1**         | IKEv1 Phase1                                              | 第一阶段：协商 IKE 管理 SA，建立加密控制通道                              |               |
| **Phase‑2**         | IKEv1 Phase2                                              | 第二阶段：在 IKE SA 隧道内协商 IPsec 业务 SA                         |               |
| **IKE_SA**          | IKE Security Association                                  | IKE 管理 SA，IKEv1 Phase1 产物；IKEv2 IKE_SA_INIT+IKE_AUTH 产物 |               |
| **CHILD_SA**        | Child Security Association                                | IKEv2 子 SA，等价 IKEv1 Phase2 的 IPsec 业务 SA                |               |

---

## 关键补充区分（考试高频）

1. **IKE SA**：用 Ci/Cr Cookie 标识；**IPsec SA**：用 `SPI + 协议(AH/ESP)+对端IP`三元组标识。
2. IKEv1 有 Phase1/Phase2；IKEv2 取消 Phase 概念，使用 `IKE_SA` + `CHILD_SA`。
3. NATT (NAT 穿越)：检测到 NAT 之后，IKE 和 ESP 全部封装 UDP4500。