---
title: 关于个人网站的仓库管理办法
excerpt: 个人网站是一个小型的项目，因此在这里记录一下仓库管理流程和办法
category: misc
tags:
  - git
cover: assets/covers/image.jpg
date: 2026-06-30
published: false
---
## 1. 固定两条长期常驻分支

| 分支名    | 对应 Vercel 环境     | 域名                       | 数据库 & 接口   |
| ------ | ---------------- | ------------------------ | ---------- |
| `dev`  | Dev 开发环境（固定预览环境） | xxx                      | 测试库、沙箱接口   |
| `main` | Production 生产环境  | https://www.simonsu.top/ | 正式业务库、真实接口 |

> 规则：

1. 所有功能分支：`feature/xxx`、`fix/xxx`，只允许合并到 `dev`；
2. `dev` 测试验收全部没问题后，再开 PR 合并进 `main`；
3. **禁止直接 push main/dev，两条分支全部开启 GitHub 分支保护，必须走 PR**。

## 2. Vercel 一对一绑定配置

1. 进入项目 → Settings → Environments
    
    - **Production 环境**：绑定分支 = `main`
        
        只要代码合并到 main，自动发布正式网站；使用生产环境变量。
    - **Preview 自定义 Dev 环境**：新增自定义环境，绑定分支 = `dev`
        
        只要 push / 合并到 dev 分支，自动部署 dev 站点；只读取 Dev 环境变量。
    
2. PR 临时预览：所有 feature 分支提交 PR，自动生成临时 Preview 预览链接，用来 UI 评审（对应你之前聊的 Preview 版本）。

### 环境变量严格隔离（绝对不能混用）

- 变量作用域分开勾选：
    
    - Production 变量：仅作用于 main 分支
    - Dev 变量：仅作用于 dev 分支
    - PR 预览变量：给 feature 临时分支使用
    

> dev 环境永远不能填写生产数据库密钥。

---

# 二、完整工作流（commit + push + PR 全流程，完全解决你之前基线冲突问题）

## 步骤 1：日常开发（所有人统一操作）

1. 永远从最新的 `dev` 分支拉取代码创建功能分支

bash

运行

```
# 拉取远端最新dev
git fetch origin
# 基于dev新建功能分支
git checkout -b feature/xxx origin/dev
```

2. 本地修改，多次 commit

bash

运行

```
git add .
git commit -m "[feat] 新增功能 #123"
```

3. 推送到远端，发起 PR 合并到 `dev`

bash

运行

```
git push origin feature/xxx
```

- GitHub 自动触发：CI 检查 + Vercel 自动构建 PR 临时预览站点；
- 同事 Code Review，问题修改后继续 push 更新 PR；
- 评审通过，合并进 `dev`。

4. 一旦合并到 dev 分支
    
    ✅ Vercel 自动重新部署 **dev 开发环境**（shturl.cc/j），测试人员在 dev 环境完整验收。

---

## 步骤 2：上线生产（dev 验收完毕才执行）

1. 在 GitHub 创建 PR：把 `dev` → 合并到 `main`
2. 再次执行回归测试，检查代码；
3. PR 合并到 main 分支
    
    ✅ Vercel 自动发布到 **Production 正式环境**（主域名）Vercel。

---

# 三、如何彻底规避你之前遇到的基线冲突（A 组合并 dev，B 分支基线落后）

## 强制两条铁律（GitHub 配置强制锁死）

### 1. 开启分支保护（dev + main 两条分支都配置）

Branch Protection Rule 勾选：

✅ Require branches to be up to date before merging

含义：如果你的 feature 分支代码落后 dev 主干，合并按钮直接灰色，**不更新基线就无法合并 PR**，从根源杜绝后继冲突。

### 2. 开发提交 PR 前必须先对齐主干

B 小组操作命令：

bash

运行

```
# 拉取最新dev代码
git fetch origin
# 把dev最新代码合并进自己的feature分支，解决冲突
git rebase origin/dev
# 推送更新PR
git push --force-with-lease
```

### 分支流转顺序（线性流转，不会乱基线）

`feature分支` → PR → `dev（Dev环境自动部署）` → PR → `main（Production自动部署）`

---

# 四、三种部署触发逻辑（自动区分环境，不用手动输命令）

1. push feature 分支 → 生成临时 PR 预览（一次性 Preview 站点）
2. 代码合并到 `dev` → 自动更新 Dev 开发环境（长期稳定测试站）
3. 代码合并到 `main` → 自动更新 Production 正式站

### 本地手动部署命令（仅调试用，正式上线全部走 Git 自动触发）

bash

运行

```
# 部署到预览/Dev环境
vercel
# 强制部署生产（仅main分支使用）
vercel --prod
```

---

# 五、精简版规范文档（直接发给团队）

1. 常驻分支只有两条：dev（测试）、main（生产）；
2. 开发必须从 dev 拉取 feature 分支，禁止直接修改 dev/main；
3. 所有改动先合入 dev，在 dev 环境测完，再提 PR 上线 main；
4. 提交 PR 前必须 rebase 对齐 dev 最新代码，保证基线一致；
5. Vercel 两套环境变量完全隔离，dev 禁止访问生产资源。

---

# 六、补充：两套方案二选一

### 方案 A（我上面这套，推荐中小团队）

双分支：dev + main，Dev 环境绑定 dev 分支，Prod 绑定 main，流程简单好维护。

### 方案 B（大厂三环境）

feature → dev（开发）→ staging（预发 Preview）→ main（Prod），多一层预发环境。

如果你需要，我可以直接导出：

1. GitHub 两条分支保护规则配置文本
2. Vercel 环境变量分环境配置清单
3. GitHub Actions 区分 dev/prod 的 yml 配置文件。