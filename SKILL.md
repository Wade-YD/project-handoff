---
name: project-handoff
description: 新项目建档与会话交接。当用户说「给某项目建档 / 初始化项目文档」「新开会话怎么快速上手这个项目」「继续上次的活 / 上次做到哪了」「写个 AGENTS.md / 交接文档 / HANDOFF」「会话要结束或中断了，记录进度」「项目文档太乱想规范化」时加载。产出三层：AGENTS.md（常驻）、docs/ARCHITECTURE.md（按需）、HANDOFF.md（滚动覆写），含冷启动验收。只产出文档、不生成代码地图（全仓分层地图用 codemap）；也不用于复盘工作流（用 reflect）。
---

# 项目建档与会话交接

## 三层结构

| 层 | 文件 | 大小上限 | 加载时机 | 内容性质 |
|---|---|---|---|---|
| 常驻入口 | `AGENTS.md` | **150 行**（20-30 行起步） | ZCode 自动加载 | 每任务都要用的 |
| 架构 | `docs/ARCHITECTURE.md` | 不设硬限 | AGENTS.md 指针引出，按需读 | 一次性摸清的 |
| 会话状态 | `HANDOFF.md` | 30-50 行 | **开工必读**，由 AGENTS.md 首节引出 | 天天变的 |

**机制事实（别想当然）**：ZCode 的工作区 AGENTS.md 从 cwd 向上逐级找，**命中即停**，到最近的含 `.git` 的祖先目录为止；`~/.zcode/AGENTS.md` 不参与这个竞争，始终叠加在最前。推论有两条，都会影响建档：

- **git 子项目读不到父目录的 AGENTS.md** → 每个项目必须自带一份
- **非 git 子目录会一路向上读到父级 AGENTS.md** → 一旦给它建档，父级规则立刻失效（包括 `.bak` 回滚约定），必须把该带的条件复制进新文件

内置 `/init` 也会写 AGENTS.md，规则是「编辑而非覆盖」。可先跑它出草稿，再按可发现性测试砍到 20-30 行。

## 四种动作

### A. 建档

0. **先盘点，不覆盖**：`ls AGENTS.md HANDOFF.md HANDOVER.md 交接文档.md docs/ARCHITECTURE.md`
   - 已有 `AGENTS.md` → **逐节 merge，禁止整体重写**，保留原有项目特有条款；冲突以原有为准，差异写进 HANDOFF 的 Open Questions
   - 已有 `HANDOFF.md` 但内容不是交接 → 先走动作 D 分流
1. **先判 git**：`git rev-parse --is-inside-work-tree >/dev/null 2>&1 || echo NO_GIT`
   - `NO_GIT` → 必须在 AGENTS.md 写死：「本项目无 `.git`：任何非平凡改动前，先把目标文件复制为 `<name>.bak-<YYYY-MM-DD>`。回滚只能靠这些副本。」HANDOFF 的 State of Play 用文件清单代替 git 状态
2. **扫，但别猜**：读 README、manifest、目录结构、CI 配置
3. **真跑命令**：build / test / lint 各跑一遍记实际结果。**不许编命令、不许留 `<...>` 占位符**；没有测试就写 `Test: none — 手工验收：<步骤>（<YYYY-MM-DD>）`
4. **写 `AGENTS.md`**（骨架见 `references/templates.md`）：只写模型读代码发现不了的
5. **写 `docs/ARCHITECTURE.md`**；**把 HANDOFF 模板复制到 `docs/HANDOFF-TEMPLATE.md`**（项目自带模板才能跨会话稳定，不依赖本 skill 是否加载）
6. **跑冷启动测试**（动作 C）

**哪些必须读代码写、哪些能扫**：目录树/入口点/构建命令/依赖清单可静态提取；架构分层/关键流程/坑**必须读代码并引 `file:line`**；边界与验收条件不可自动，只能问人。

**不建档**：单次脚本、<10 个文件的实验目录、纯数据目录。

### B. 写 handoff

1. **先核实**：`git status --short --branch`（一条给分支 + ahead/behind + 脏文件；unborn 分支给 `## No commits yet on main`）。无 git 项目改用 `ls -lt` 看最近改动
2. 按 `references/templates.md` 的 6 段**覆写** `HANDOFF.md`（同名覆写，**不新开日期文件**）
3. 两段最高价值别省：**`Learnings & Landmines`**（已排除的死路 + 原因 + 禁改区）、**`What We Tried`**（时间序；失败方案是最贵的重发现）
4. `Next Steps` 第一条必须是可直接复制的字面命令或 `file:line`
5. **禁止写密钥、token、连接串、账号**——HANDOFF 要入库，只写 `.env.local` 的键名

### C. 冷启动测试（建档后必跑）

当前上下文里已经有刚读过的代码，**自己判卷无效**。改为：**新开一个会话或派子智能体**，只给项目路径、不给任何提示，让它回答：

1. 这项目解决什么问题？跑起来要几条命令？
2. 要加一个 X 功能，从哪个文件开始看？
3. 什么改动会搞坏已安装副本 / 用户数据 / 线上（按项目实际择一）？哪些区域禁碰？
4. 怎么确认改动是对的？
5. `HANDOFF.md` 的 Next Steps 第一条是什么？

**答不上 = 文档没写完。** 这同时验证了「只给地址就能上手」这个最终目标。

### D. 迁移旧交接文档

1. **先读，别改名**。先判断它属于哪一类
2. 按类落位：

| 旧文件的实际内容 | 去向 |
|---|---|
| 当前进度、未完成项、下一步 | 压成新的 `HANDOFF.md`（≤50 行） |
| 架构、模块地图、语义基准 | `docs/ARCHITECTURE.md` |
| 设计交付说明（定位 / 视觉 / API 契约） | `docs/DESIGN.md`，**不要放 HANDOFF.md** |
| 「本次会话修复 N 项」流水账 | 只抽仍有效的结论与死路进 Learnings，原文移入 `docs/history/` 或删除 |

3. **取链头**：旧文档顶部若指向更晚的 `docs/交接-YYYY-MM-DD-*.md`，**当前状态在那份里**，先顺着指针读完再压。直接改名会把过期内容变成唯一入口
4. 旧文件用 `git mv` 改名退役；全项目只留一个 `HANDOFF.md`

## 写作硬规则

- **可发现性测试**：写每节前问「agent 读代码能不能自己发现？」能，就别写。**目录树、技术栈、数据模型摘要一律删**
- **每条断言引 `file:line`**：写不出引用的 gotcha 就是噪音，删掉
- **写正面规则，不写禁令**——**但只作用于 AGENTS.md 的规则区**。HANDOFF 的 Learnings 与 ARCHITECTURE 的 Danger zones **必须点名**死路和禁改区：它们附带原因和替代路径，没有替代方案的禁令才是反模式
- **20-30 行起步，150 行是天花板**（经验值：指令越多依从性越衰减，且每行都在和真正的任务抢注意力）
- **`UNVERIFIED` 必须带验证路径**：孤立的标签等于一个沉默的未知
- **每条规则只能写在一个文件里**，其他文件用指针

## 反模式

- **会话流水账**（「本次修复 17 项」）——那是日志，不是交接
- **设计交付说明冒充 handoff**——`HANDOFF.md` 回答的是「下一步做什么」
- **多套命名并存**（`HANDOVER.md` / `交接文档.md` / `HANDOFF.md`）——一个项目一份，滚动覆写
- **复述 README 或目录结构**——零信息增量
- **文档写完就不管**——HANDOFF 的「最后核实」日期早于最近一次 commit 即视为过期，先按 git 重写 State of Play 再动手

## 参考

- `references/templates.md` —— 三份文件的完整可抄模板
- `references/writing-rules.md` —— 每条规则的**为什么**与出处（含明确排除的工具）
