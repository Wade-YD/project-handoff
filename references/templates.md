# 三层文件 + 一份按需挂账的可抄模板

> **三层必建**（`AGENTS.md` / `docs/ARCHITECTURE.md` / `HANDOFF.md`）**+ 一份按需**（`docs/backlog.md`，只有真实发现时才建）。`AGENTS.md`、`HANDOFF.md` 落仓库根，其余落 `docs/`。本文的 `AGENTS.md` 一律指 `<仓库根>/AGENTS.md`（项目级），**不是** `~/.zcode/AGENTS.md`（全局个人偏好）。

## 一、`<仓库根>/AGENTS.md`（项目级常驻入口，≤150 行）

```markdown
# <项目名>

<一段：这系统是什么、干什么、给谁用。从代码来，不是愿景。>

## 开工第一件事
1. 读 `HANDOFF.md`（≤60 行）。它不是背景资料，是上次会话的断点。
2. 跑 `git status --short --branch` 核对（无 git 项目：`ls -lt` 看最近改动）。
3. 两者不一致时以仓库为准：先按 `docs/HANDOFF-TEMPLATE.md` 覆写 `HANDOFF.md`，再动手。
4. `HANDOFF.md` 的「最后核实」日期早于最近一次 commit → 先按 git 重写 State of Play。

## Commands
- Run: `<命令>`
- Test: `<命令>` — passes on clean checkout (<YYYY-MM-DD>)
  没有测试就写：`Test: none — 手工验收：<命令 + 预期输出>（<YYYY-MM-DD>）`
- Lint: `<命令>`

## Ground rules
<只写有 2+ 例子能证明的约定，每条引两个位置>
- <约定>（`path/a.ts:12`, `path/b.ts:30`）
- INCONSISTENT: <两种并存写法> —— 动手前问用户，别擅自加第三种

## Gotchas
<≤3 条。每条必须引 file:line（且回读核对过该行内容）或 "observed running <cmd>"。写不出引用就别写。>
- <坑>（`path/file.ts:88`）

## Done means
- 有 CI 就照抄 CI 里的 verify 命令；没有 CI 就写「跑通 `<命令>` 且看到 `<可观察输出>`」
- 任何行为变更都带一个「不带这个改动就失败」的测试

## 本项目的约束
<无 .git 时必写，有 .git 时删掉本节>
- 本项目无 `.git`：任何非平凡改动前，先把目标文件复制为 `<name>.bak-<YYYY-MM-DD>`。回滚只能靠这些副本。

## 深入
- 改任何模块前先看 `docs/ARCHITECTURE.md`（Router Table、Danger zones、一条真实 feature 的完整追踪）。
- 未闭合缺陷与验收缺口：`docs/backlog.md`（有则读，无则不必建）。

## 收工前
用户说「先到这」/ 准备 commit / 上下文快满时：按 `docs/HANDOFF-TEMPLATE.md` 覆写 `HANDOFF.md`，同名覆写，不新开日期文件。
本次发现的缺陷/缺口先落 `docs/backlog.md`，再覆写 HANDOFF。
```

**删掉这些**（agent 自己能发现，写了纯占注意力）：目录树、技术栈清单、数据模型摘要、复述文件布局的架构说明。

---

## 二、`docs/ARCHITECTURE.md`（按需读）

**缩减规则**：仓库 <20 个源文件、或交付物是单文件 HTML → 省略本文件，Components 并入 AGENTS.md 且 ≤5 行；`HANDOFF.md` 仍然必须有。

```markdown
# <项目名> · 架构

## Where to look for X（Router Table，10-15 行）
| 要改什么 | 从哪看起 |
|---|---|
| <功能/模块> | `path/to/file.ts:42` |

## Danger zones
- <看着安全实则有毒的区域>（`file:line`）—— <为什么危险>

## Components
| 组件 | 职责 | 状态 | 证据 |
|---|---|---|---|
| <名> | <做什么> | works / half-wired / dead | `path:line` |

## How a feature is added here
<挑一个最近真实改过的特性，从头到尾追一遍：
 入口 → 调用链 → 副作用 → 终态，每步带文件名。>
<这是全篇最有价值的一节——新人照这条路径走，不要走通用模式。>

## Data flow
<写路径 AND 读路径，分步带文件名。只写一条会漏掉一半的坑。>

## Gotchas
- <竞态 / 初始化顺序 / 隐性耦合>（`file:line`）

## Unverified
- <没验证的推断> —— 用 `<命令>` 可以证实
```

**`状态` 列怎么判定**（别猜，按探针来）：

```bash
rg -n "<组件名>" --glob '!**/*.md' --glob '!**/*.test.*'
```

- 只出现在测试 / 文档 / 注释 → `dead`
- 有生产调用但没接到真实入口（没挂载、没注册路由、没写进配置、开关默认关）→ `half-wired`，证据列写**缺口位置**
- 能从入口跑到并观察到真实结果 → `works`，证据写「跑 `<命令>` 得到 `<输出摘要>`」
- **拿不准填 `half-wired`，并在 `Unverified` 写验证命令。不要猜 `works`。**

---

## 三、`HANDOFF.md`（会话状态，每次覆写；**≤60 行**，`wc -l` 计含空行，30 行起步）

> 文件名随项目既有约定（`HANDOVER.md` 等，见 SKILL.md 动作 A 第 0 步「既有命名优先」）；本节统一以 `HANDOFF.md` 指代，落盘时全局替换为项目实际名。

```markdown
# HANDOFF

**最后核实:** <YYYY-MM-DD> · 分支 `<branch>`（用 `date +%F` 取当天，pwsh 用 `Get-Date -Format yyyy-MM-dd`，不许手填；早于最近一次 commit 即视为过期）

**TL;DR:** <1-3 句：这是什么、到哪一步、唯一的下一步动作>

## Intent & Goal
<为什么做、终态是什么、验收标准（最好是那条要通过的命令或测试）>

## State of Play（已对 git 核实；无 git 项目：已对最近改动文件清单核实）
- Branch: `<branch>`
- Committed: <已提交的改动>
- Uncommitted: <未提交的改动，或「无」>
- 已验证可用: <真跑过的>
- 只是假设未验证: <没跑过的>
- 无 git 项目：用最近改动的文件清单代替以上四项

## What We Tried
<时间序：试了什么 → 结果 → 为什么弃用。
 失败方案是最贵的重发现，后来的 agent 不该重走一遍。>

## Learnings & Landmines
- <非显然的发现>
- 死路：<方案> —— <为什么不行>
- 禁改区：`file:line` —— <原因>

## Pointers
- `path/to/file.ts:123` —— <这行是干嘛的>
- <相关文档 / 复现命令>

## Next Steps
1. <字面动作：可直接复制的命令或 file:line>
2. <...>

## Open Questions
- <未决问题> —— 当前倾向：<...>
```

**四条硬规则**：

1. 写完前必须跑过 `git status --short --branch`。**基于聊天记录写的、自信但错误的 handoff，比没有更糟。**
2. **禁止写密钥、token、连接串、账号**——HANDOFF 要进 git；只写 `.env.local` 里的键名。
3. **活干完就退役**：目标达成后压成 5 行（结论 + 唯一遗留项）或删除，不保留已完成的历史交接——否则它迟早退化成第二份流水账。
4. **缺陷不进本文件**：审查/建档发现的问题落 `docs/backlog.md`；HANDOFF 只留一行指针，Next Steps 被某条阻塞时引它。**超 60 行（`wc -l` 计）按序砍**：① 与 AGENTS.md 重复的 Pointers ② Open Questions（迁 backlog「待确认」区）③ Intent & Goal 压成一句 ④ What We Tried 只留死路与原因——**State of Play 的核实事实、Learnings、Next Steps 第一条不砍**。

---

## 四、`docs/HANDOFF-TEMPLATE.md`

把上面第三份的 fenced markdown 块原样复制进去（「四条硬规则」是行为约束，留在本 skill 与项目 AGENTS.md 的「收工前」节，不进模板文件）。建档时必须创建它——项目自带模板才能跨会话稳定，不依赖 skill 是否被加载。

---

## 五、`docs/backlog.md`（缺陷与验收缺口挂账，**按需创建**）

> 只在有真实发现时创建；没有就留空，**不要建一个空文件**。已有 `docs/backlog.md` / `TODO.md` 一类文件就续写，**不要新建第二个**（Windows 大小写不敏感，`BACKLOG.md` 与 `backlog.md` 会互相覆盖）。

```markdown
# Backlog（未闭合缺陷与验收缺口）

> 写入约定：新发现只进「未闭合」；修完从「未闭合」移除，在「已闭环」压成一行。
> 「来源」必填——写不清来源的条目必须标 `待核实`，不得当已核实事实。

## 未闭合（按 P0→P3 排序，一屏能扫完）

- [P1] <一句话问题>（`path:line`）— 影响：<什么场景错 / 丢什么>｜来源：<审查名或会话 + 日期>｜去向：open
- [P2] <...>｜来源：<...>，**仓内无落盘出处，待核实**

## 待确认（不是缺陷，是缺信息）

- <问题> —— 确认方式：<问用户 / 跑哪条命令>

## 不修（含理由，防止反复被提出）

- <问题> —— 不修理由：<...>

## 已闭环（一行一条，留结论不留过程）

- <YYYY-MM-DD> [P1] <问题> —— 修法：<一句话>（`path:line`）
```

**四条写入纪律**：

1. **未闭合超过约 15 条**时，先合并同类、把确认不修的移入「不修」——**只增不减即视为该文件已腐化**。
2. 每条必须能让人 60 秒内判断「要不要现在动手」；写不出影响面的，是噪音。
3. P0/P1 挡住当前工作的，HANDOFF 的 `Next Steps` **引条目、不复制描述**。
4. **「来源」是必填项**——这正是「结论只活在会话记忆里」那次事故的根因。
