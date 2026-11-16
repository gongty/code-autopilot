# code-autopilot

**写代码、做项目的自动驾驶：一个总控带 N 条线并行开发，巡检纠偏、审查合并，不撞额度。**

**Autopilot for coding projects in Claude Code and Codex: one controller runs parallel coding sessions, keeps them on track, reviews and merges, and stays within usage limits.**

![License: MIT](https://img.shields.io/badge/license-MIT-blue) ![Claude Code](https://img.shields.io/badge/Claude%20Code-skill-d97757) ![Codex](https://img.shields.io/badge/Codex-skill-black)

[中文](#中文) · [English](#english)

---

## 中文

code-autopilot 是一个给 Claude Code 和 Codex 用的 Agent Skill（`SKILL.md`）。

接到复杂的开发需求（从 0 到 1 做新项目、跨几个模块的大功能、重构），它让当前对话当总控，把活拆成几条线。每条线是一个独立会话，有自己的工作副本（git worktree）和分支，全自动推进，你睡觉时也能接着做。

### 能做什么

- **总控是顶级架构师、项目总负责人**：为整个结果负责，要效果最好、token 最省。它定方案、排顺序、写任务书、验收、合并，自己不写业务代码；查找和审查交给子 agent。
- **开工一次对齐**：要你定的（方案谁定、什么时候要、设计怎么交、哪些事要你点头）开工时一条消息问完，每项带推荐，回一句「都按推荐」就开工。
- **开发原则**：第一性原理定结构；举一反三，从源头改，不打补丁；代码越少越好；只复用热门又极简的开源项目；依赖没定死不开工；测试适度，节点上集中测；文档和记忆跟着代码走。
- **设计线**：要界面的，开一条线用 Claude Design 出稿。界面只引用一份 `tokens.css`，交活附截图对稿，对过才合，做出来和稿子不会两个样。
- **按难度派模型**：要判断的给强模型，照单执行的给便宜一档；Fable 很贵，要用先问你。
- **巡检**：每 20 分钟看一次，卡住、原地打转、超时的当场纠正。
- **额度管家**：默认开着。每次巡检查 5 小时和周额度，推算到重置时会不会超；不够就让线停在一个提交上，定时等额度恢复再接着做。
- **合并门要快**：只跑受影响的测试，每个测试文件有硬上限，门排队，同一份代码不重跑。

### 实测

2026-11-15 到 16，作者用一个总控对话先后带 7 条线做完一次完整重构。

16 号夜里 5 条线没人盯：01:12 开线，09:46 上线，周额度用了约 20 个点，5 小时额度没撞墙。读数和踩过的坑在 `references/quota.md`、`references/review.md`。

### 前提

- 首选 Claude Code 桌面端，这套就是在那里跑出来的。命令行、Codex 是照官方文档写的，还没在真项目里跑过。
- 总控请设成 Opus 5.5、思考强度超高、权限自动（auto）。
- 默认会开 2–5 条线，额度大约 2–5 倍地烧。

### 安装

整个仓库就是一个 skill，克隆进 skill 目录就装好了：

```bash
git clone https://github.com/gongty/code-autopilot ~/.claude/skills/code-autopilot
```

| 用什么 | 克隆到哪 |
| --- | --- |
| Claude Code，所有项目 | `~/.claude/skills/code-autopilot` |
| Claude Code，只给一个项目 | `<项目>/.claude/skills/code-autopilot` |
| Codex | `~/.agents/skills/code-autopilot`（Codex 不读 `.claude/skills`） |
| 腾讯 WorkBuddy | `~/.workbuddy/skills/code-autopilot`（还没真跑过，见 `references/workbuddy.md`） |

更新：在那个目录里 `git pull`。

### 用法

装好就默认生效：接到复杂的开发需求就走这套，一两个文件的小改动照常直接做。

也可以直接说：「这个项目你当总控，全自动推进」。

### 常见问题

#### 和子 agent、agent teams 有什么不同？

子 agent 的结果回到总控、占总控的上下文，用户也打不开它；agent teams 的队友共用一个目录，没有各自的工作副本。

这里的每条线是完整的独立会话：自己的工作副本和上下文，做完还能接着派活，你也能点进去直接说话。

#### 会不会把额度烧光？

开 N 条线大约 N 倍地烧。

额度管家每次巡检推算到重置时的用量，超过 85% 就让离关键路径最远的线停在一个提交上，重置后定时叫醒。

#### 能整夜没人盯吗？

能，实测过一晚。

要开着桌面端、接电源、别合盖，权限设成自动（auto），不然线会停下来等你点「允许」。

#### 能用在 Codex、WorkBuddy 吗？

同一份 `SKILL.md` 都能读，工具叫法的对照在 `references/tools.md`。WorkBuddy 开不了新对话，替代办法在 `references/workbuddy.md`。

两边都还没在真项目里跑过。

#### 小改动也会开线吗？

不会。一两个文件、半小时内改得完的，直接做。

### 文件

| 文件 | 写什么 |
| --- | --- |
| `SKILL.md` | 总控的全部规矩 |
| `references/line-rules.md` | 每条线要守的 |
| `references/patrol.md` | 巡检、停手、续上、纠正四段提示词 |
| `references/quota.md` | 额度推算和一晚上的真实读数 |
| `references/review.md` | 验收、审查、合并，合并门怎么提速 |
| `references/design.md` | 设计线和对稿 |
| `references/tools.md` | 各环境的工具对照、权限 |
| `references/workbuddy.md` | WorkBuddy 里怎么用 |
| `references/sources.md` | 官方文档和社区做法的出处 |
| `templates/kickoff.md`、`plan.md` | 开工对齐、开发计划 |
| `templates/project.md`、`brief.md`、`notes.md`、`design-brief.md` | 项目卡、任务书、总控备忘、设计总纲 |
| `templates/audit-prompt.md` | 项目梳理提示词，防止项目变成屎山 |

---

## English

code-autopilot is an Agent Skill (`SKILL.md`) for Claude Code and Codex.

For complex development work (a new project from zero, a feature spanning several modules, a refactor), it turns the current conversation into a controller that splits the work into lines. Each line is a separate session with its own git worktree and branch. The work moves forward on its own, overnight included.

The skill is written in Chinese; the agent reads it either way and replies in your language.

### What it does

- **The controller is a top-tier architect and the project owner**: it is responsible for the result and aims for the best outcome at the lowest token cost. It plans, orders the work, writes briefs, reviews and merges, and writes no product code itself; searching and reviewing go to subagents.
- **One kickoff message**: everything you need to decide (who owns the plan, deadline, design delivery, what needs your approval) is asked once, with a recommendation for each. Reply "all as recommended" and it starts.
- **Principles**: structure from first principles; fix the class of problem at its source, no patches; less code is better; reuse only popular, minimal open-source projects; nothing starts before its dependencies are settled; test in proportion, fully at merge and cutover points; docs and memory move with the code.
- **Design line**: for anything with a UI, a line designs it in Claude Design. UI code uses one `tokens.css`; deliveries come with screenshots checked against the design before merging.
- **Model per task**: strong models where judgment is needed, a cheaper tier for step-by-step briefs. Fable is expensive and needs your approval.
- **Patrols**: every 20 minutes it catches lines that are stuck, looping or over time, and corrects them.
- **Usage manager**: on by default. It reads 5-hour and weekly usage on every patrol and projects it to the next reset. If the work won't fit, lines pause at a commit and a timer resumes them after the reset.
- **Fast merge gate**: affected tests only, a hard time limit per test file, one gate at a time, and the same tree is never gated twice.

### Field-tested

On 2026-11-15/16 the author used one controller conversation to lead 7 lines through a complete refactor.

On the night of the 16th, 5 lines ran unattended: lines started at 01:12, the release went out at 09:46, about 20 points of weekly usage were used, and the 5-hour limit was never hit. Readings and lessons are in `references/quota.md` and `references/review.md`.

### Requirements

- Claude Code desktop (Code tab) is the primary environment, where this was developed. The CLI and Codex instructions follow the official docs and have not run on a real project yet.
- Set the controller to Opus 5.5, effort xhigh, permission mode auto.
- It runs 2–5 lines by default, which uses usage roughly 2–5 times as fast.

### Install

The whole repo is one skill. Clone it into a skills folder:

```bash
git clone https://github.com/gongty/code-autopilot ~/.claude/skills/code-autopilot
```

| Tool | Clone to |
| --- | --- |
| Claude Code, all projects | `~/.claude/skills/code-autopilot` |
| Claude Code, one project | `<project>/.claude/skills/code-autopilot` |
| Codex | `~/.agents/skills/code-autopilot` (Codex does not read `.claude/skills`) |
| Tencent WorkBuddy | `~/.workbuddy/skills/code-autopilot` (not run yet, see `references/workbuddy.md`) |

Update with `git pull` in that folder.

### Usage

Once installed it is the default for complex development requests; small changes to one or two files are still done directly.

You can also say: "Be the controller for this project and drive it end to end."

### FAQ

#### How is this different from subagents or agent teams?

Subagent results flow back into the controller's context and you can't open them; agent-team teammates share one directory without their own worktrees.

Each line here is a full session with its own worktree and context. It can take the next task when done, and you can open it and talk to it.

#### Will it burn through my usage limits?

N lines use roughly N times the usage.

On every patrol it projects usage to the next reset. Above 85%, it pauses the lines furthest from the critical path at a commit and wakes them after the reset.

#### Can it run overnight unattended?

Yes, tested over one night.

Keep the desktop app open, plugged in, lid open, and use auto permission mode, or lines will stop and wait for you to click "allow".

#### Does it work with Codex or WorkBuddy?

Both read the same `SKILL.md`; tool mappings are in `references/tools.md`. WorkBuddy can't open new conversations; the workaround is in `references/workbuddy.md`.

Neither has run on a real project yet.

#### Does it open lines for small changes?

No. Changes to one or two files that take under half an hour are done directly.

### Files

| File | Contents |
| --- | --- |
| `SKILL.md` | All controller rules |
| `references/line-rules.md` | Rules for every line |
| `references/patrol.md` | Patrol, pause, resume and correct prompts |
| `references/quota.md` | Usage projection and one night of real readings |
| `references/review.md` | Review, merge, and a faster merge gate |
| `references/design.md` | Design line and design-to-code checks |
| `references/tools.md` | Tool mapping per environment, permissions |
| `references/workbuddy.md` | Using it in WorkBuddy |
| `references/sources.md` | Official docs and community sources |
| `templates/kickoff.md`, `plan.md` | Kickoff message, development plan |
| `templates/project.md`, `brief.md`, `notes.md`, `design-brief.md` | Project card, line brief, controller notes, design brief |
| `templates/audit-prompt.md` | Project audit prompt that finds structural root causes |

---

## 许可 License

MIT，见 `LICENSE`。 MIT, see `LICENSE`.
