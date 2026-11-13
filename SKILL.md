---
name: project-handoff
description: 新项目建档与会话交接。当用户说「给某项目建档 / 初始化项目文档」「新开会话怎么快速上手这个项目」「继续上次的活 / 上次做到哪了」「写个 AGENTS.md / 交接文档 / HANDOFF」「会话要结束或中断了，记录进度」「项目文档太乱想规范化」时加载。产出三层：**项目级** `<仓库根>/AGENTS.md`（常驻，非全局 `~/.zcode/AGENTS.md`）、`docs/ARCHITECTURE.md`（按需）、`HANDOFF.md`（滚动覆写），含冷启动验收。只产出文档、不生成代码地图（全仓分层地图用 codemap）；也不用于复盘工作流（用 reflect）。
---

# 项目建档与会话交接

## 三层结构

> ⚠️ **全部三个文件都放「项目根目录」，不是用户级目录。** 本 skill 说的 `AGENTS.md` 一律指 `<仓库根>/AGENTS.md`（项目级，随仓库走），**绝不要写进 `~/.zcode/AGENTS.md`**——那是全局个人偏好，对所有项目生效，塞项目专属内容会污染每个会话。

| 层 | 文件 | 大小上限 | 加载时机 | 内容性质 |
|---|---|---|---|---|
| 常驻入口 | **`<仓库根>/AGENTS.md`**（项目级） | **150 行**（20-30 行起步） | ZCode 自动加载 | 每任务都要用的 |
| 架构 | `<仓库根>/docs/ARCHITECTURE.md` | 不设硬限 | 项目 AGENTS.md 指针引出，按需读 | 一次性摸清的 |
| 会话状态 | **`<仓库根>/HANDOFF.md`** | **60 行**（`wc -l` 计，含空行；30 行起步） | **开工必读**，由项目 AGENTS.md 首节引出 | 天天变的 |

**机制事实（别想当然）**：ZCode 的工作区 AGENTS.md 从 cwd 向上逐级找，**命中即停**，到最近的含 `.git` 的祖先目录为止；**全局 `~/.zcode/AGENTS.md` 不参与这个竞争，始终叠加在最前**。所以每份项目级 AGENTS.md 都是「全局规则 + 本项目规则」的叠加结果——只写本项目特有的，别复述全局已有的。推论有两条，都会影响建档：

- **git 子项目读不到父目录的项目级 AGENTS.md** → 每个项目必须自带一份
- **非 git 子目录会一路向上读到父级 AGENTS.md** → 一旦给它建档，父级规则立刻失效（包括 `.bak` 回滚约定），必须把该带的条件复制进新文件

内置 `/init` 也会写项目级 AGENTS.md，规则是「编辑而非覆盖」。可先跑它出草稿，再按可发现性测试砍到 20-30 行。

## 四种动作

### A. 建档

0. **先盘点，不覆盖**：`ls AGENTS.md HANDOFF.md HANDOVER.md 交接文档.md docs/ARCHITECTURE.md`
   - 已有 `AGENTS.md` → **逐节 merge，禁止整体重写**，保留原有项目特有条款；冲突以原有为准，差异写进 HANDOFF 的 Open Questions
   - 已有 `HANDOFF.md` 但内容不是交接 → 先走动作 D 分流
1. **先判 git**：`git rev-parse --is-inside-work-tree >/dev/null 2>&1 || echo NO_GIT`
   - `NO_GIT` → 必须在 AGENTS.md 写死：「本项目无 `.git`：任何非平凡改动前，先把目标文件复制为 `<name>.bak-<YYYY-MM-DD>`。回滚只能靠这些副本。」HANDOFF 的 State of Play 用文件清单代替 git 状态
2. **扫，但别猜**：读 README、manifest、目录结构、CI 配置
3. **真跑命令，跑通的才算门禁**：build / test / lint 各跑一遍记实际结果，把**原文命令**抄进 AGENTS.md `Commands`，并在 `Done means` 指明合并前必须绿的那一条（多条用 `&&` 串成一行）。**不许编命令、不许留 `<...>` 占位符**；没有自动化测试就写 `Test: none — 手工验收：<命令 + 预期输出>（<YYYY-MM-DD>）`，同时到 `docs/backlog.md` 记一条「无自动化验收」。本 skill 只负责把闸门**固化并执行**，不负责设计测试体系
4. **写 `<仓库根>/AGENTS.md`（项目级）**（骨架见 `references/templates.md`）：只写模型读代码发现不了的；**勿写进全局 `~/.zcode/AGENTS.md`**
5. **写 `docs/ARCHITECTURE.md`**；**把 HANDOFF 模板复制到 `docs/HANDOFF-TEMPLATE.md`**（项目自带模板才能跨会话稳定，不依赖本 skill 是否加载）
6. **缺陷有落点**：建档/审查中发现的问题（死按钮、假成功、必崩命令、无验收）逐条登记到 `docs/backlog.md`（模板见 `references/templates.md` §五）。已有 `docs/backlog.md` / `TODO.md` 一类文件就续写，**不新建第二个**
7. **跑冷启动测试**（动作 C）

**哪些必须读代码写、哪些能扫**：目录树/入口点/构建命令/依赖清单可静态提取；架构分层/关键流程/坑**必须读代码并引 `file:line`**；边界与验收条件不可自动，只能问人。

**不建档**：单次脚本、<10 个文件的实验目录、纯数据目录。

### B. 写 handoff

1. **先核实**：`git status --short --branch`（一条给分支 + ahead/behind + 脏文件；unborn 分支给 `## No commits yet on main`）。无 git 项目改用 `ls -lt` 看最近改动
2. **先跑一次 `Done means` 的闸门**：只跑增量成本可接受的；需要全量重编/联网的写「未跑 + 原因 + 命令」，不得写成已验证。结果进 State of Play，并注明是否跑在含在途改动的树上——如实记录，不归因
3. 按 `references/templates.md` 的 6 段**覆写** `HANDOFF.md`（同名覆写，**不新开日期文件**）
4. 两段最高价值别省：**`Learnings & Landmines`**（已排除的死路 + 原因 + 禁改区）、**`What We Tried`**（时间序；失败方案是最贵的重发现）
5. `Next Steps` 第一条必须是可直接复制的字面命令或 `file:line`
6. **引用回读**：覆写后把全文 `file:line` 抽出来逐条回读该行，确认描述与该行内容一致（`grep -nE '[[:alnum:]_./-]+:[0-9]+' HANDOFF.md` 可抽清单）
7. **超限按序砍**：① 与 AGENTS.md 重复的 Pointers ② Open Questions（迁 backlog「待确认」）③ Intent & Goal 压成一句 ④ What We Tried 只留死路与原因。**State of Play 的核实事实、Learnings、Next Steps 第一条不砍**
8. **禁止写密钥、token、连接串、账号**——HANDOFF 要入库，只写 `.env.local` 的键名

### C. 冷启动测试（建档后必跑）

当前上下文里已经有刚读过的代码，**自己判卷无效**。改为派**读取型子智能体**（ZCode 用 `Explore`；其他工具用等价的只读 agent，**不要用 general-purpose / 通用型**），prompt 只给绝对路径 + 下面 5 问，并明确写：

> 只允许读项目文档（`AGENTS.md` / `HANDOFF.md` / `docs/ARCHITECTURE.md` 及其指针文件）。
> **不要执行任何命令，不要读 `.env*`，不要扫描 `node_modules` / `dist` / `target` 等生成目录。**
> 答不上就判文档不合格，**不要替我跑测试或到代码里找答案**。

1. 这项目解决什么问题？跑起来要几条命令？
2. 要加一个 X 功能，从哪个文件开始看？
3. 什么改动会搞坏已安装副本 / 用户数据 / 线上（按项目实际择一）？哪些区域禁碰？
4. 怎么确认改动是对的？
5. `HANDOFF.md` 的 Next Steps 第一条是什么？

**答不上、或靠读代码才答上 = 文档没写完。** 测试的目的是证明「只给地址就能上手」，不是替你做验收——闸门实跑属于动作 A/B。

### D. 迁移旧交接文档

1. **先读，别改名**。先判断它属于哪一类
2. 按类落位：

| 旧文件的实际内容 | 去向 |
|---|---|
| 当前进度、未完成项、下一步 | 压成新的 `HANDOFF.md`（≤60 行） |
| 架构、模块地图、语义基准 | `docs/ARCHITECTURE.md` |
| 设计交付说明（定位 / 视觉 / API 契约） | `docs/DESIGN.md`，**不要放 HANDOFF.md** |
| 「本次会话修复 N 项」流水账 | 只抽仍有效的结论与死路进 Learnings，原文移入 `docs/history/` 或删除 |

3. **取链头**：旧文档顶部若指向更晚的 `docs/交接-YYYY-MM-DD-*.md`，**当前状态在那份里**，先顺着指针读完再压。直接改名会把过期内容变成唯一入口
4. 旧文件用 `git mv` 改名退役；全项目只留一个 `HANDOFF.md`

## 写作硬规则

- **可发现性测试**：写每节前问「agent 读代码能不能自己发现？」能，就别写。**目录树、技术栈、数据模型摘要一律删**
- **每条断言引 `file:line`，引用必须回读核对**：先 Read 该行，再把该行的关键标识符（函数名 / 常量 / 文案原文）写进断言；写不出引用、或引用与该行内容不符的 gotcha 都是噪音——**引对行号却说反内容，比没有引用更危险**
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
- **缺陷只活在会话记忆里**——审查/建档发现的每条问题都必须落 `docs/backlog.md` 且**写上来源**；只写进 HANDOFF 会在下次覆写时消失
- **backlog 只增不减**——没有「修完移出、压成一行」的纪律，它三个月后就是一个没人读的坟场

## 参考

- `references/templates.md` —— 三层文件的完整可抄模板 + `docs/backlog.md` 挂账模板（§五）
- `references/writing-rules.md` —— 每条规则的**为什么**与出处（含明确排除的工具）
