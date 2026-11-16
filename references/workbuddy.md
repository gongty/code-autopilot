# 在腾讯 WorkBuddy 里用

查于 2026-11-16。依据是作者电脑上 WorkBuddy AI 自带的 agent-cli 2.165.0 程序本身（下面标「程序里核到」）和官方文档。**还没在 WorkBuddy 里真跑过一条线**，第一次用照最后一节验一遍。

## 和 Claude Code 差在哪

Claude Code 是三层：总控 → 线 → 线自己的子 agent。每条线都是一个完整的对话：侧栏看得见，你能点进去直接说话，它能自己再开子 agent，总控关了它照跑。

WorkBuddy 的总控开不了新对话。能自动开的只有「后台队友」（`Agent` 加 `run_in_background: true`），和 Claude Code 的线比：

| | Claude Code 的线 | WorkBuddy 的后台队友 |
| --- | --- | --- |
| 你点进去说话 | 能 | 不能，只能经总控转（`SendMessage` 进它的信箱） |
| 自己的工作副本 | 开线时就给 | 不给，和总控共用目录。`isolation: worktree` 只对不在后台跑的子 agent 生效（程序里核到） |
| 再往下开 | 能开子 agent | 不能再开队友（程序原文）；能不能开普通子 agent 查东西：没验证 |
| 总控对话关了 | 照跑 | 一起停：队友跑在总控这个进程里 |
| 线的输出 | 留在它自己的对话里 | 回到总控的上下文，几条线就能把总控撑满 |

所以分两种开法，按线的大小选，也可以混着用。

## 开法甲：自动开线（后台队友）

适合任务书写到能照单执行、半小时到一小时的线。

1. **先放好线的定义**：项目里 `.codebuddy/agents/code-autopilot-line.md`。要自己判断边界、设计方案的线另放一个 `code-autopilot-core.md`，`effort: high`。
   ```markdown
   ---
   name: code-autopilot-line
   description: code-autopilot 的执行线：照任务书在指定的工作副本里写代码、跑测试、提交
   model: <WorkBuddy 里便宜一档的模型>
   effort: medium
   ---
   你是线：不用 code-autopilot，不开新线。照任务书做，守 <主目录>/.code-autopilot/briefs/common.md。
   你和总控共用一个目录：所有文件路径都用任务书第一行那个工作副本下的绝对路径，每条命令都以 `cd <工作副本> &&` 开头。
   进度和最后的报告写进 <主目录>/.code-autopilot/reports/<线名>.md；回给总控的话不超过三行：做到哪、提交号、卡在哪。
   ```
   frontmatter 能写 `model`、`effort`（minimal / low / medium / high / xhigh / max）、`maxTurns`、`tools`、`skills`（程序里核到）。
2. **总控替线建工作副本**：`git worktree add <主目录>-wt/<线名> -b code-autopilot/<线名> <基线>`。放仓库外面，免得测试工具扫进去；`.env` 这类被 gitignore 的文件要自己拷过去。
3. **开线**：`Agent`，`subagent_type: code-autopilot-line`，`name: <线名>`，`run_in_background: true`。prompt 只写一句：「你是线 <线名>。工作副本 <绝对路径>。任务书 <绝对路径>，先读它。」
4. **看线、发话、停线**：`ListAgents` 看哪些在跑；`TaskOutput` 看一次某条的输出（看一次就好，别循环等）；`SendMessage` 发进它的信箱；`TaskStop` 或 `InterruptAgent` 停。
5. 同时最多 2–3 条：都挤在一个进程里，上限没验证。

## 开法乙：手动开线（你自己开新对话）

适合大的线：要自己再开子 agent 查找和审查的、你想点进去看的、要跑好几个小时的。三层结构和 Claude Code 一样，代价是开线和传话不自动。

1. 总控照常建工作副本、写任务书，然后给你一段开场白：「你是线 <线名>，不用 code-autopilot。先读 <主目录>/.code-autopilot/briefs/common.md（线的规矩）和任务书 <路径>。工作副本 <路径>。报告写进 <主目录>/.code-autopilot/reports/<线名>.md；每次提交后读一眼 <主目录>/.code-autopilot/inbox/<线名>.md，那是总控给你的话。」
2. 你在 WorkBuddy 新开一个任务，工作目录选那个工作副本，把开场白粘进去。
3. 总控给这个对话发不了消息。要纠正、要停，总控写进 `<主目录>/.code-autopilot/inbox/<线名>.md`，线每次提交后看一眼（开场白里写了）；急的让你转贴。
4. 巡检照常，只是看的是文件：reports、`git log`、进程（`patrol.md` 里那张表）。

## 其余几样怎么对

| | WorkBuddy 里 | 要点 |
| --- | --- | --- |
| 装 skill | `~/.workbuddy/skills/code-autopilot/`（所有项目），或项目里 `.codebuddy/skills/code-autopilot/` | 程序里核到会扫这两处 |
| 项目规矩 | 读 `CODEBUDDY.md`，也读 `AGENTS.md` | |
| 选模型 | 能选的模型和 Claude 不同，照 SKILL.md 第三节的思路：总控用最强的，照单执行的用便宜一档，思考强度中 | |
| 定时巡检 | 工具表里有 `CronCreate` / `CronList` / `CronDelete` | 桌面端到点会不会触发：没验证。不触发，就在你回来时说一声「巡检」 |
| 额度 | 按积分扣；程序里没找到让 agent 读剩余积分的工具 | 开工前请你看一眼剩余积分、给这批活定个上限，记进 `notes.md`；第一条线跑完算出每小时大约烧多少，之后照 SKILL.md 第六节推算，到上限的 85% 停线 |
| 权限 | 默认就是全放开（bypass，程序注释原文） | 黑名单更要紧：删真实数据、读密钥写进 deny（推主分支别写：总控和线在同一个进程，总控合完也推不上去；线不推主分支写在线的规矩里）。**计费的生成工具（出图、出视频这类）不管什么模式都弹窗等你批**：别派给线，要用就在你在场时由总控调 |
| 上下文 | 有自动压缩；窗口怎么设：没验证 | 总控到七八成先写 `notes.md`，照旧 |
| 撞额度、关应用 | 后台队友跟着总控一起停 | 规矩不变：线随时停在一个提交上、报告写进文件。重开后新开一条线，先读报告和 `git log` 接着做 |

## 第一次用，先验这四样

1. 装好后新开对话，问它有哪些 skill，看到 code-autopilot。
2. 开一条后台队友，让它跑一次 `pwd` 和 `cd <工作副本> && git branch --show-current`，确认路径规矩管用、没改到主目录。
3. 建一个 2 分钟后的 `CronCreate`，看到点会不会触发。
4. 跑完看积分扣了多少，填进 `notes.md`。

验完把这一页的「没验证」改成实测结果。
