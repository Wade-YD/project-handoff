# project-handoff · 项目建档与会话交接 Skill

[English](README.en.md) | **简体中文**

一个给 AI 编程 agent（Claude Code / ZCode / Codex / OpenCode 等）用的 Skill：**新开会话只给一个项目路径，agent 就能快速上手**——把散落在各处的 `HANDOFF.md` / `HANDOVER.md` / `交接文档.md` 收敛成一套三层文档结构，并自带建档、交接、验收、旧文档迁移四个动作。

## 解决什么问题

- 新会话摸代码要从零开始；交接文档写了却**不会被自动加载**（agent 只自动读 `AGENTS.md`）
- 交接文档质量参差：有的是设计交付说明，有的是「本次会话修复 17 项」的流水账，多套命名并存
- 生态调研结论：`anthropics/skills`（19 个官方 skill）与 `obra/superpowers`（15 个）均无 handoff/onboarding 类；[agents.md](https://agents.md/) 标准明确「无必填字段、无官方模板」——这个空缺由本 skill 填补

## 三层结构

> ⚠️ **三个文件都放「项目根目录」，不是用户级目录。** 本文的 `AGENTS.md` 一律指 `<仓库根>/AGENTS.md`（**项目级**，随仓库走），**不是** `~/.zcode/AGENTS.md` / `~/.claude/CLAUDE.md`（全局个人偏好）——塞项目专属内容进去会污染每个会话。

| 层 | 文件 | 大小上限 | 加载时机 | 内容性质 |
|---|---|---|---|---|
| 常驻入口 | **`<仓库根>/AGENTS.md`**（项目级） | **150 行**（20-30 行起步） | agent 自动加载 | 每个任务都要用的 |
| 架构 | `<仓库根>/docs/ARCHITECTURE.md` | 不设硬限 | 项目 AGENTS.md 指针引出，按需读 | 一次性摸清的 |
| 会话状态 | `<仓库根>/HANDOFF.md` | 30-60 行 | **开工必读**，由项目 AGENTS.md 首节引出 | 天天变的 |

关键机制：项目 `AGENTS.md` 只放「每任务都要用」的东西（全局偏好已在 `~/.zcode/AGENTS.md`，**别复述**——ZCode 会把两者叠加注入）；架构细节用指针挂到懒加载层；会话状态单独一个文件**滚动覆写，不新开日期文件**——否则会重演多套命名散落的问题。

## 四种动作

| 动作 | 触发说法 | 做什么 |
|---|---|---|
| **A 建档** | 「给这个项目建档」 | 先盘点已有文档（**逐节 merge，禁止覆盖**）→ 真跑 build/test → 写三层文档 → 过冷启动测试 |
| **B 写 handoff** | 「写个 handoff / 会话要结束了」 | 先跑 `git status` 对仓库核实 → 6 段覆写 `HANDOFF.md` |
| **C 冷启动测试** | 建档后必跑 | **新开会话或派子智能体**，只给路径不给提示，回答 5 个上手问题；答不上 = 文档没写完 |
| **D 迁移旧文档** | 「把交接文档规范化」 | 按内容分流（进度→HANDOFF / 架构→ARCHITECTURE / 设计说明→DESIGN.md / 流水账→只抽结论），**先顺着链头指针读完最新状态再压** |

## 写作硬规则（skill 内置，防文档腐烂与噪音）

- **可发现性测试**：agent 读代码能自己发现的就别写——目录树、技术栈、数据模型摘要一律删
- **写正面规则，不写禁令**（点名废弃 API 会让模型更想用它）；但 HANDOFF 的死路记录必须点名
- **150 行是天花板**：指令越多依从性越衰减，每一行都在和真正的任务抢注意力
- **每条 gotcha 引 `file:line`**：写不出引用的就是噪音；`UNVERIFIED` 必须带验证路径
- **handoff 写完前必须跑 `git status`**：基于聊天记录写的、自信但错误的交接，比没有更糟

## 安装

```bash
# 方式一：skills CLI
npx skills add <owner>/project-handoff

# 方式二：手动复制（通用）
git clone https://github.com/<owner>/project-handoff.git
cp -r project-handoff ~/.agents/skills/     # ZCode / OpenCode
# 或 cp -r project-handoff ~/.claude/skills/  # Claude Code
```

## 使用

对 agent 说：

```
用 project-handoff 给 <项目路径> 建档
```

```
写个 handoff，我要收工了
```

> 语言说明：`SKILL.md` 与 references 正文为中文，AI agent 可直接执行；issue / PR 中英皆可。

## 不同 agent 的 AGENTS.md 加载机制（建档前必看）

| 工具 | 加载规则 | 对建档的影响 |
|---|---|---|
| ZCode | 从 cwd 向上找**最近一个**，到最近含 `.git` 的祖先为止，不合并父级 | git 子项目读不到父级规则；**给非 git 目录建档会切断父级规则**（如有 `.bak` 回滚约定需一并复制进新文件） |
| Claude Code | root → cwd **全部拼接**；2.1.277 起支持 AGENTS.md（无 CLAUDE.md 时兜底） | 单文件建议 <200 行 |
| Codex | 逐级向下拼接，合计上限 **32 KiB** | 各层文件要控制总量 |
| OpenCode | AGENTS.md 优先；`opencode.json` 的 `instructions` 可外挂任意 md | 唯一支持外挂 HANDOFF 的 |

## 实战验证

首个试点：一个 ~1.1k 行的 Python + pywebview 桌面工具（私有项目）。

- 冷启动测试通过：全新会话只给路径，五问全答对（解决什么/从哪看起/禁碰区/怎么验收/下一步）
- 建档与验证过程**反向抓出该项目 8 个既有缺陷**（含一个用户可见的假成功、一个必崩命令、三处 GUI 死按钮）与 1 个流程缺陷
- skill 产出的文档自身也被冷启动测试抓出 1 处事实错误并自纠——**文档错了测试能抓住**，这正是冷启动测试存在的意义

## 设计来源与致谢

本 skill 的骨架拼自三个独立来源，各取所长：

| 来源 | 借了什么 |
|---|---|
| [johnpapa/ai-ready](https://github.com/johnpapa/ai-ready) | 可发现性测试、正面规则、150 行天花板、`Done means` |
| [silkyland/know-my-repo](https://github.com/silkyland/know-my-repo) | `file:line` 强制、`UNVERIFIED` 带验证路径、冷启动测试 |
| [blader/baton](https://github.com/blader/baton) | 6 段 handoff 结构、git 核实铁律、「错误交接比没有更糟」 |
| [REMvisual/claude-handoff](https://github.com/REMvisual/claude-handoff) | `What We Tried`（失败方案是最贵的重发现） |
| [cathrynlavery/repo-atlas](https://github.com/cathrynlavery/repo-atlas) | 架构层取舍（10 份砍到 4 段）、文档腐烂检测思路 |

明确排除：`aider --repo-map`（不落盘）、`repomix` / `code2prompt`（是打包器不是文档生成器）。

## License

[MIT](LICENSE)
