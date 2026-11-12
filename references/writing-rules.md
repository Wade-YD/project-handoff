# 为什么是这些规则（防「顺手简化」）

规则本身在 `SKILL.md` 和 `references/templates.md`，这里只记**理由与出处**。删规则前先读这里。

## 可发现性测试 —— `johnpapa/ai-ready`

agent 读代码能自己发现的就别写。**目录树、技术栈、数据模型摘要全部删掉**——它们一个月就过期，且每行都在抢注意力。

留下的四类**不可发现**信息：构建/测试命令（埋在 tooling 配置里）、布局为什么反常、从 review 里挖出的约定、维护矩阵（改了 A 必须同步改 B）。

## 写正面规则 —— `ai-ready` + `mattpocock/skills · writing-for-agents`

写 `Use the v2 client in lib/api/`，不写 `Do not use the v1 client`。**点名废弃的东西会让模型更容易去用它。** 两个独立来源互相印证。

**作用域限定**：这条只管 AGENTS.md 的规则区。HANDOFF 的 Learnings 和 ARCHITECTURE 的 Danger zones **必须点名死路**——它们带原因和替代路径，没有替代方案的禁令才是反模式。

## 150 行天花板 —— `ai-ready`

从 20-30 行起步。经验值是模型遵循约 150-200 条指令后依从性开始衰减，且上下文预算是有限的。**这是经验值，不是精确科学**，但方向对：AGENTS.md 长到读不完就等于没有。

## `file:line` 强制 —— `silkyland/know-my-repo`

写不出引用的 gotcha 就是噪音。真没法引用时写 `observed running <cmd>`，但必须真跑过。

配套的 **`UNVERIFIED` 必须带验证路径**：一个孤零零的 UNVERIFIED 标签、没有验证路径，等于一个沉默的未知。

## 冷启动测试 —— `know-my-repo`

写完文档后只读文档扮成新 agent，回答开头列的问题；答不上就算没写完。**注意**：当前上下文里已经有刚读过的代码，自测无效——必须新开会话或派子智能体。

## handoff 必须对 git 核实 —— `blader/baton`

写完前跑 `git status --short --branch`。**基于聊天记录写的、自信但错误的 handoff，比没有更糟。**

## 死路是最高价值信息 —— `baton` + `REMvisual/claude-handoff`

两个独立来源都把「已排除的死路」列为最有价值内容：**失败方案是最贵的重发现**。合并成一个时间序的 `What We Tried` + 结论式的 `Learnings & Landmines` 即可，不要两份都留。

## 一个项目一份，滚动覆写 —— `baton`

日期文件会散落成一堆命名不一的交接文档，后来者不知道读哪个。需要历史就用 `git` 管，不用文件名管。**活干完压成 5 行或删**，否则它退化成第二份流水账。

## 明确排除的工具

不是文档生成器，别浪费时间：

- `aider --repo-map`：只在内存里给模型看，**不落盘**
- `repomix` / `code2prompt`：上下文喂料 / 打包器
- `agentrules-architect` 这类多模型流水线：要 API key + 6 阶段，单个本地小项目不划算

## 唯一正面解决「文档腐烂」的设计 —— `cathrynlavery/repo-atlas`

`make atlas-check`：自动生成的部分进 CI 校验，人工部分靠 review。值得借鉴思路，但不必上 10 份文档。

没有 CI 时的替代：HANDOFF 的「最后核实」日期早于最近一次 commit 即视为过期。
