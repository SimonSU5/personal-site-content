---
title: 个人网站部署方案
excerpt: 我原先使用vercel部署我的个人网站，和github关联，静态无后端，部署容易。但是我最近给我的个人网站添加了一个后台管理功能，vercel不能做对象存储，不支持写入，而且国内访问相当慢（套cf代理估计也不能加速）。因此打算购入一个阿里云的ecs+oss+cdn，重新部署了我的个人网站。这篇博客记录一下部署方案
category: misc
tags:
  - OSS
  - ECS
  - CDN
  - git
  - GitHub-action
cover: assets/covers/个人网站部署方案-封面.png
date: 2026-07-04
published: false
---
# 阿里云 E 实例 (2 核 2G)+OSS+CDN+Nginx 部署个人网站

我原先使用vercel部署我的个人网站，和github关联，静态无后端，部署容易。但是有以下两个问题：

1. 我最近给我的个人网站添加了一个后台管理功能，vercel不能做对象存储，不支持写入，而且国内访问相当慢（套cf代理估计也不能加速）。
2. 我的后台管理功能是前端实现，因此每次更改内容需要改前端代码。这就意味着功能实现的代码和内容的代码会因此购入了一个阿里云的oss，重新部署了我的个人网站。这篇博客记录一下过程。

**整体架构分层**：用户浏览器 → 阿里云 CDN → ECS Nginx 网关 → OSS 对象存储
**配套 CI/CD**：GitHub Actions 云端构建、分版本原子发布、零停机更新、双环境 (dev/main) 隔离。

## 一、架构分层与每一层职责（核心底层逻辑）
### 1. 最外层：CDN（流量承载核心，约95% 流量）

1. **缓存分层**
    - 边缘节点：全国 300 + 节点，用户就近访问 JS/CSS/ 图片 / 字体，命中后直接返回，不回源 ECS；
    - 缓存规则区分两类文件：
        - 带哈希静态资源（`app.32fd12.js`）：设置`max-age=31536000, immutable`，永久缓存；
        - `index.html`：不缓存`no-cache`，每次强制回源，保证拿到最新页面入口。
    
2. **回源逻辑**
    CDN 只在两种情况访问你的 2 核 2G ECS：
    - 用户首次访问、缓存过期、主动刷新缓存 → 请求`index.html`；
    - 缓存未命中的新静态资源。
    
3. **流量分摊优势**
    图片、视频、大资源由 CDN 扛，ECS 仅处理 html 路由转发。

### 2. 中间层：E 实例 2 核 2G Nginx 网关

#### 反向代理、路由、SSL、缓存控制，**不存储任何静态文件**

1. Nginx 资源占用
    
    - Nginx 是多进程异步 IO 模型，master 进程管理 worker，worker 进程单线程处理上万并发；
    - 纯反向代理场景：常驻内存仅 20~40MB，CPU 空闲占用＜5%；
    - 系统 Ubuntu/CentOS 基础占用 300~400MB，整机总内存占用不会超过 600MB，2G 内存余量巨大。
    
2. 两大核心功能
    
    （1）SSL 终止：443 端口 HTTPS 解密 / 加密，证书存在 ECS 本地，握手逻辑由 Nginx 完成，OSS 不直接暴露公网 HTTPS；
    （2）SPA 前端路由兜底：history 模式刷新 404 拦截，统一转发`index.html`；
    （3）分环境域名隔离（main和dev分支见我“个人网站仓库管理办法”一文）：
    - `simonsu.top` → 代理 OSS `releases/prod/latest/`（生产 main 分支）
    - `shturl.cc/S` → 代理 OSS `releases/dev/latest/`（测试 dev 分支）
    
3. 零停机重载原理
    `nginx -s reload` 不会杀死现有连接：
    - 启动新 worker 进程加载新配置；
    - 旧 worker 处理完存量用户请求后自动销毁；
    - 全程无连接断开，实现和 Vercel 一致的无缝切换。

### 3. 底层存储层：OSS 对象存储（真正存放前端 dist 产物）

#### 核心设计：分版本目录 + latest 软链接 实现原子发布（零停机关键）

1. 目录规范底层逻辑

plaintext

```
bucket/
├─ releases/
   ├─ prod/
   │  ├─ a7f29d/  # main分支某次commit哈希，完整静态包
   │  ├─ e8c124/  # 上一版稳定包
   │  └─ latest/  # 软链接目录，指向当前线上活跃版本
   └─ dev/
      ├─ b3d561/  # dev分支测试包
      └─ latest/
```

2. 原子发布完整流程（解决上传过程页面残缺问题）
    
    错误做法：直接覆盖`latest`，上传一半时用户访问会出现新旧文件混杂、404；
    
    标准原子流程：
    
    ① GitHub Actions 先把完整 dist 上传独立 commit 哈希目录（全量上传完成前，线上 latest 完全不动）；
    
    ② 全部文件上传校验成功后，**删除旧 latest，完整复制新版本目录到 latest**；
    
    ③ 刷新 CDN 缓存，Nginx 配置无需改动；
    
    全程线上用户访问旧版本，切换瞬间完成，无断层。
3. OSS 内网优势
    
    ECS 和 OSS 同阿里云账号同地域，内网访问 OSS 不计公网流量，反向代理回源速度更快、不消耗服务器带宽。

## 二、CI/CD GitHub Actions 自动化构建技术细节（替代 Vercel 内置部署）

### 1. 流水线触发规则（完全对齐你 dev/main 分支流程）

yaml

```
on:
  push:
    branches: [main, dev] # 仅合并到两条常驻分支自动部署
```

- push 到`dev`：打包上传`releases/dev/短哈希/`，更新 dev/latest，刷新 dev 二级域名 CDN；
- push 到`main`：打包上传`releases/prod/短哈希/`，更新 prod/latest，刷新主域名 CDN。

### 2. 流水线分步底层执行逻辑

1. `actions/checkout`：拉取当前分支完整代码；
2. `setup-node`：安装 Node 环境，`npm ci`精准锁定 package-lock 依赖，避免版本差异；
3. `npm run build`：前端打包（Vite/Webpack），输出 dist 静态资源，自动生成带 contenthash 的资源文件名；
    
    - 哈希文件名核心作用：新旧版本资源共存 CDN，旧页面不会加载不到旧 JS/CSS，无白屏；
    
4. 获取 commit 短哈希：`git rev-parse --short HEAD`，作为版本唯一标识；
5. ossutil 工具上传：云端直接和 OSS 内网通信，不经过你的 ECS；
6. 原子替换 latest 目录，调用 CDN 刷新接口。

### 3. 和 Vercel 部署的底层等价性

- Vercel：构建在 Vercel 云端，每个分支生成独立部署实例，域名原子切换；
- 这套方案：构建在 GitHub 云端，每个 commit 生成 OSS 独立版本目录，latest 软链接原子切换；
    
    两者底层发布逻辑完全一致，只是存储载体从 Vercel 平台换成阿里云 OSS。

## 三、双环境隔离技术细节（dev 测试 /main 生产完全隔离）

### 1. 三层隔离机制，测试操作绝不污染线上

1. **存储隔离**
    
    dev 代码全部存在`releases/dev/`，生产在`releases/prod/`，两套目录物理独立，互不干扰；
2. **域名隔离**
    
    Nginx 两套 server 块：
    
    - `shturl.cc/S` 指向 dev 目录，仅内部测试人员访问；
    - `shturl.cc/F` 指向 prod 目录，对外公网用户；
    
3. **环境变量隔离**
    
    GitHub Actions 区分分支注入环境变量：
    
    - main 分支打包注入生产接口、正式密钥；
    - dev 分支打包注入测试沙箱接口、测试数据库地址；
        
        打包时直接替换前端环境变量，两套产物完全独立，不会出现 dev 包调用线上接口。
    

### 2. 访问权限管控（可选）

dev 二级域名可在 CDN 配置 IP 白名单，仅公司内网可访问，外部用户无法进入测试环境。

## 四、零停机发布底层完整时序（拆解每一步用户访问状态）

以 main 分支合并新版本为例：

1. 合并代码到 main → GitHub Actions 启动流水线；
2. 云端打包 dist，完整上传 OSS `releases/prod/a7f29d/`；
    
    👉 此时用户访问仍走旧`prod/latest`，线上完全不受影响；
3. 全量文件上传完成，校验文件大小无缺失；
4. 原子替换`prod/latest`指向新版本 a7f29d；
5. 调用 CDN 刷新接口，清除 CDN 内旧 index.html 缓存；
6. 用户行为：
    
    - 新用户：CDN 拉取新 index.html，加载新版本；
    - 正在浏览的存量用户：继续使用 CDN 缓存的旧页面，会话结束后刷新才会加载新版；
    
7. 回滚操作：只需将 latest 重新指向旧 commit 哈希目录，同样零停机恢复。

## 五、SPA 路由 404 问题底层解决方案（两套兜底）

### 1. OSS 静态网站托管兜底

OSS 后台开启静态网站，404 转发至`index.html`，当 Nginx 异常时作为备用；

### 2. Nginx 反向代理强制兜底（主方案）

nginx

```
location / {
    proxy_pass https://xxx.oss-cn-hangzhou.aliyuncs.com/releases/prod/latest/$uri;
    error_page 404 https://xxx.oss-cn-hangzhou.aliyuncs.com/releases/prod/latest/index.html;
}
```

原理：前端 history 路由刷新时，OSS 找不到对应路径返回 404，Nginx 拦截 404 响应，替换为首页 index.html，前端路由接管地址，不会出现空白页。

## 六、2 核 2G ECS 性能边界与资源调度细节

### 1. 资源消耗拆解

- 系统基础（Ubuntu 22.04）：350MB 内存；
- Nginx 常驻进程：30MB 内存；
- certbot 定时证书续期（每月一次短时执行）：瞬时占用 CPU＜10%；
- 剩余可用内存：1.5G+，CPU 长期空闲；

### 2. 并发承载上限

单 Nginx 反向代理可支撑**单秒 1000 + 请求**，对应日 PV 3~5 万以内完全无压力；

### 3. 性能瓶颈规避关键点

1. 不要在这台 ECS 部署后端服务、数据库、Redis；数据库单独购买阿里云 RDS；
2. CDN 务必开启，否则大量静态资源回源会占用 ECS 带宽；
3. Nginx 开启 gzip 压缩，减少传输流量，降低服务器负载。

## 七、故障回滚技术细节（两种安全方案）

### 方案 1：业务回滚（多人协作标准，无历史污染）

1. GitHub 原 PR 页面点击 Revert，生成反向提交 PR；
2. 合并 Revert PR 到 main，Actions 自动打包旧逻辑、上传 OSS 新版本目录；
3. latest 自动切换，完成回滚，Git 历史完整留存，可审计。

### 方案 2：紧急手动回滚（线上突发故障秒级恢复）

1. 登录阿里云 ossutil/OSS 控制台；
2. 将`releases/prod/latest`目录重新复制为上一个稳定 commit 哈希文件夹；
3. 刷新 CDN 缓存，全程不改动 ECS，30 秒内恢复线上旧版本。

## 八、整套架构对比 Vercel 底层差异总结

表格

|维度|Vercel|阿里云 OSS+2 核 2G ECS|
|---|---|---|
|构建环境|Vercel 云端容器|GitHub Actions 云端容器|
|静态存储|Vercel 自有分布式存储|阿里云 OSS 对象存储|
|流量分发|Vercel 全球边缘 CDN|阿里云国内 CDN|
|网关服务|Vercel 托管反向代理|自建 Nginx 网关（2 核 2G ECS）|
|版本机制|每次部署独立实例|OSS 按 commit 哈希分版本目录|
|零停机原理|原子切换域名指向实例|原子切换 latest 目录软链接|
|自定义能力|有限 Nginx 配置|完整自定义 Nginx、限流、重定向、IP 管控|
|成本|高流量付费贵|按量计费，静态站长期成本更低|

## 九、关键技术避坑底层原理

1. **禁止直接覆盖 latest 目录**
    
    上传过程中目录文件不完整，用户访问会出现 JS 缺失、页面错乱；必须先上传独立版本，再整体替换 latest。
2. **index.html 不能强缓存**
    
    若设置长缓存，用户浏览器会长期缓存旧首页，发布新版本后用户刷新无变化；静态资源带哈希可永久缓存。
3. **不要用 ECS 本地存放 dist 文件**
    
    本地存储会存在磁盘 IO、版本备份、扩容问题，OSS 无限容量、自动多副本备份，更适合静态资源。
4. **reload 和 restart 区别**
    
    `nginx -s restart`会断开所有现有用户连接，造成瞬间断站；`nginx -s reload`平滑切换，生产环境只能用 reload。