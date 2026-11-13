# project-handoff · Project Onboarding & Session Handoff Skill

**English** | [简体中文](README.md)

A Skill for AI coding agents (Claude Code / ZCode / Codex / OpenCode, etc.): **start a new session with nothing but a project path, and the agent gets up to speed** — it consolidates scattered `HANDOFF.md` / `HANDOVER.md` / session-log files into a three-layer documentation structure, with four built-in actions: bootstrap, handoff, acceptance, and legacy migration.

## The problem it solves

- New sessions re-discover the codebase from scratch; handoff docs exist but are **never auto-loaded** (agents only auto-read `AGENTS.md`)
- Handoff docs drift in quality: some are design delivery specs, some are "fixed 17 items this session" session logs, with inconsistent naming
- Ecosystem research: neither `anthropics/skills` (19 official skills) nor `obra/superpowers` (15) has a handoff/onboarding skill; the [agents.md](https://agents.md/) spec explicitly defines "no required fields, no official template" — this skill fills that gap

## Three layers

> ⚠️ **All three files live in the project root — not in a user-level directory.** `AGENTS.md` here always means `<repo-root>/AGENTS.md` (**project-level**, travels with the repo), **not** `~/.zcode/AGENTS.md` / `~/.claude/CLAUDE.md` (global personal preferences) — polluting the global file with project-specific content affects every session.

| Layer | File | Size cap | Loaded | Content |
|---|---|---|---|---|
| Resident entry | **`<repo-root>/AGENTS.md`** (project-level) | **150 lines** (start at 20-30) | auto-loaded by the agent | what every task needs |
| Architecture | `<repo-root>/docs/ARCHITECTURE.md` | uncapped | via pointer in the project AGENTS.md, on demand | one-time understanding |
| Session state | `<repo-root>/HANDOFF.md` | 30-60 lines | **required reading at session start** | what changes daily |

Key idea: the project `AGENTS.md` holds only what every task needs (global preferences already live in `~/.zcode/AGENTS.md` — **don't restate them**, agents stack both); architecture details hang off a pointer in the lazy layer; session state lives in one file that is **rolled over in place — never dated copies**, which is exactly how naming sprawl happens.

> Additionally `docs/backlog.md` — **created on demand, not part of the three layers** (audit findings land there instead of living only in session memory).

## Four actions

| Action | Trigger phrase | What it does |
|---|---|---|
| **A Bootstrap** | "set up docs for this project" | Inventory existing docs first (**merge per-section, never overwrite**) → actually run build/test → write the three layers → pass the cold-start test |
| **B Handoff** | "write a handoff / session ending" | Verify against `git status` → overwrite `HANDOFF.md` in 6 sections |
| **C Cold-start test** | mandatory after bootstrap | **A fresh session or subagent**, given only the path, must answer 5 onboarding questions; any miss = docs unfinished |
| **D Migrate legacy docs** | "normalize our handoff docs" | Route by content (progress→HANDOFF / architecture→ARCHITECTURE / design specs→DESIGN.md / session logs→extract conclusions only); **follow chain-head pointers to the latest state before compressing** |

## Writing rules built in (against doc rot and noise)

- **Discoverability test**: if the agent can find it by reading code, don't write it — directory trees, tech-stack lists, and data-model summaries are all banned
- **Positive rules, not prohibitions** (naming a deprecated API makes models more likely to use it) — except dead-end records in HANDOFF, which must be named
- **150 lines is a ceiling**: instruction-following decays with volume; every line competes with the real task
- **Every gotcha cites `file:line` — and the citation must be re-read**: a right line number with the wrong description is worse than no citation
- **Run `git status` before finishing a handoff**: a confident, wrong handoff written from chat memory is worse than none
- **Defects must not live only in session memory**: audit/bootstrap findings go to `docs/backlog.md` (**created on demand**, not a fourth layer), source is mandatory
- **Gates get recorded and executed**: if there are no tests, write `Test: none + manual acceptance command` and log the gap in backlog — **never force the agent to invent a test framework**

> Note: `docs/backlog.md` is created on demand and is not part of the three layers — only build it when there are real findings.

## Install

```bash
# Option 1: skills CLI
npx skills add <owner>/project-handoff

# Option 2: manual copy (universal)
git clone https://github.com/<owner>/project-handoff.git
cp -r project-handoff ~/.claude/skills/      # Claude Code
# or cp -r project-handoff ~/.agents/skills/  # ZCode / OpenCode
```

## Usage

Tell your agent:

```
Use project-handoff to bootstrap docs for <project path>
```

```
Write a handoff, I'm done for today
```

> Language note: `SKILL.md` and references are written in Chinese; AI agents execute them fine. Issues and PRs in either language are welcome.

## AGENTS.md loading behavior per agent (read before bootstrapping)

| Tool | Loading rule | Impact on bootstrap |
|---|---|---|
| ZCode | nearest `AGENTS.md` walking up from cwd, stopping at the closest `.git` ancestor; no parent merging | git sub-repos never see parent rules; **bootstrapping a non-git dir cuts it off from parent rules** (copy any `.bak` rollback conventions into the new file) |
| Claude Code | concatenates root → cwd; supports AGENTS.md since 2.1.277 (fallback when no CLAUDE.md) | keep each file under ~200 lines |
| Codex | concatenates level by level, total cap **32 KiB** | watch the combined size |
| OpenCode | AGENTS.md first; `opencode.json` `instructions` can mount any md | the only one that can externally mount HANDOFF.md |

## Battle-tested

First pilot: a ~1.1k-line Python + pywebview desktop tool (private project).

- Cold-start test passed: a brand-new session, given only the path, answered all five onboarding questions
- The bootstrap + verification process **surfaced 8 pre-existing defects** in that project (including a user-visible fake success, a command that always crashed, and three dead GUI buttons) plus 1 workflow defect
- The docs the skill produced had 1 factual error of their own — caught by the cold-start test and fixed. **The test catches wrong docs**; that is why it exists

## Sources & credits

The skeleton is composed from three independent schools, taking the best of each:

| Source | What was borrowed |
|---|---|
| [johnpapa/ai-ready](https://github.com/johnpapa/ai-ready) | discoverability test, positive rules, 150-line ceiling, `Done means` |
| [silkyland/know-my-repo](https://github.com/silkyland/know-my-repo) | `file:line` enforcement, `UNVERIFIED` with verification paths, cold-start test |
| [blader/baton](https://github.com/blader/baton) | 6-section handoff, git-verification rule, "a wrong handoff is worse than none" |
| [REMvisual/claude-handoff](https://github.com/REMvisual/claude-handoff) | `What We Tried` (failed approaches are the most expensive rediscovery) |
| [cathrynlavery/repo-atlas](https://github.com/cathrynlavery/repo-atlas) | architecture-layer pruning (10 docs → 4 sections), doc-rot detection ideas |

Explicitly excluded: `aider --repo-map` (never hits disk), `repomix` / `code2prompt` (packers, not doc generators).

## License

[MIT](LICENSE)
