# 出处：官方和社区怎么说，和这套做法哪里一样、哪里不同

查于 2026-11-16。「已验证」= 读到了官方页面原文；「没验证」= 只看到转述或页面之间说法打架。

## 这套做法比官方多出来的（从真项目里摸出来的）

官方文档给了零件（会话、工作副本、子 agent、定时、额度读数），没给怎么把它们组成一套能整夜无人值守跑的流程。这套补的是：

1. **总控当架构师和项目总负责人**：开工一次问清要用户定的；定方案、排顺序、定死接口、写任务书、验收、合并；自己不写业务代码。官方最接近的是「先写 SPEC 再开新会话照着做」「另开一个会话审」。
2. **估时按模型速度、上限 1.5 倍**：实测三条线的估时比实际长 4–9 倍（`../SKILL.md` 第四节）。官方没有这条。
3. **巡检的卡住标准**：15 分钟没输出、CPU 满着不出结果或很低却不退出、反复交同一份报告、同一失败修两次、超上限。官方只有「同一处纠正两次还错就清空重来」。
4. **额度管家**：每次巡检读额度，推到重置时超过 85% 就按离关键路径的远近停线、停在一个提交上；一开始就不够的线先开好等重置；撞墙前就建好叫醒的定时。官方只告诉你在哪看额度、撞了会怎样。
5. **审查要准、发回要收敛**：审查写死改动范围、测试范围和时限，不通读仓库、不跑整套；问题一次全发回，会拦合并的立刻修，互相牵连的攒一批修；同一块审出两次错就按结构发回，第三次停止补、重新设计（`review.md`）。
6. **桌面端的开线限制和对策**：自动叫醒的那一轮不能开新线 → 用户发话那轮一次开好、后面的活发给空着的线（`tools.md`）。
7. **设计线和衔接**：新项目先由总控写设计总纲和分批任务，开一条线专门出稿（Claude 里用 Claude Design；别的环境本地写网页稿、截图自查、派子 agent 审，设计规矩取自 Claude Design 的做法，见 `design-local.md`），存进 `design/` 给研发照做；交法（直接出、先挑方向、每批都看）由用户选；总控管一张屏和代码的对照表，界面只引用一份 `tokens.css`，交活附截图、派子 agent 对稿，对过才合（`design.md`）。
8. **开发原则**：先找热门又极简的开源项目、弄清用什么加什么改什么、拿它的测试当验收；按依赖排顺序，依赖没定死不开工；不打补丁，从一个问题看到一类问题；代码越少越好；测试适度，节点上集中测；文档和记忆跟着代码走（`../SKILL.md` 开头和第一节）。来自作者做项目的经验，不是官方说法。
9. **门要快**：只跑受影响的测试、每个测试文件有硬上限、线和合流用同一个门、门排队、过了门的同一份代码不重跑（`review.md`）。

## 官方能补强这套的（已并进去的打了 ✓）

| 官方说法 | 出处 | 怎么用 |
| --- | --- | --- |
| ✓ 子 agent 新版默认能再往下开三层，`CLAUDE_CODE_MAX_SUBAGENT_SPAWN_DEPTH` 改 | https://code.claude.com/docs/en/sub-agents ｜已验证（Claude Code 2.1.322 里也有这个设置；2026-11-15 的记录里，总控的一个子 agent 真的又开了一个查找用的子 agent） | 「子 agent 不能再开子 agent」已经不对了。改成讲为什么要往下派的活还是开成线 |
| ✓ 每个子 agent 可单独写 `model` 和 `effort`；查找类用便宜档（官方举的是 Haiku） | 同上；https://code.claude.com/docs/en/model-config ｜已验证 | 第三节表格 |
| ✓ 上下文都对了还错，是模型不够，换大模型；max 容易想过头 | https://claude.com/resources/articles/claude-model-and-effort-level-in-claude-code ｜已验证 | 第三节。官方另一半「漏文件、没跑测试就调高思考强度」没照搬：作者的经验是先查任务书写没写清，强度开高了反而爱扩大范围 |
| ✓ 命令行交互会话撞上 5 小时额度，重置后自己接着做，最多 3 次；周额度、后台、`-p` 不会 | https://code.claude.com/docs/en/interactive-mode ｜已验证 | `quota.md`「撞墙了」。桌面端会不会自己接：没验证；2026-11-15 撞上组织花费上限那次没有自己接 |
| ✓ 状态栏 JSON 有 `rate_limits.five_hour / seven_day` 的百分比和重置时刻（Pro / Max / Ultra） | https://code.claude.com/docs/en/statusline ｜已验证 | `tools.md` |
| ✓ 定时只在会话空着时触发，循环的会晚最多半个周期（封顶 30 分钟），7 天过期，错过不补 | https://code.claude.com/docs/en/scheduled-tasks ｜已验证 | 第五节、`tools.md` |
| ✓ 开 10 个后台会话，额度大约是 1 个的 10 倍；多 agent 的 token 约是聊天的 15 倍 | https://code.claude.com/docs/en/agent-view ；https://www.anthropic.com/engineering/multi-agent-research-system ｜已验证 | 第六节「N 倍地烧」 |
| ✓ 「卡住」先排除在等权限批准；常用权限先放开 | https://code.claude.com/docs/en/agent-view ｜已验证 | 第零节「开工前备好」、第五节 |
| ✓ auto 模式：分类器先审再执行；连拒 3 次或累计 20 次就退回逐条问人；ask 规则照样弹 | https://code.claude.com/docs/en/permission-modes ｜已验证 | 第零节「开工前备好」；线的规矩「被拒了别重试同一件事」 |
| ✓ 别的会话发来的消息不能替用户批准任何东西 | https://code.claude.com/docs/en/cross-session-messaging ｜已验证 | 所以权限要在开线前放开，总控批不了 |
| ✓ 工作副本读主目录的 `settings.local.json`；`--permission-prompts none` 让没人应答的直接拒 | https://code.claude.com/docs/en/worktrees 、/headless ｜已验证（Claude Code 2.1.322 有这个参数，只对 `-p` 生效） | `tools.md` 权限一节 |
| ✓ 同一处纠正两次还错，清空上下文、把教训写进新提示重来 | https://code.claude.com/docs/en/best-practices ｜已验证 | 第五节：重新设计后开一条上下文干净的线 |
| ✓ 规划、生成、评审分开，模型给自己打分总偏高 | https://www.anthropic.com/engineering/harness-design-long-running-apps ｜已验证 | 和「审查员不是作者」一致 |
| ✓ 长任务靠进度文件 + git 提交交接，每次开工先读进度和 git log | https://www.anthropic.com/engineering/effective-harnesses-for-long-running-agents ｜已验证 | 和「停在一个提交上」「notes.md」一致 |
| 功能清单用 JSON（带 `passes` 字段），比 Markdown 不容易被模型乱改 | 同上 ｜已验证 | 没并进去：这套的验收靠测试和总表，不靠清单。要用可以在项目卡里加 |
| 线做完用 `notify_when_idle` 通知、`claude agents --json` 看状态，定时巡检只当兜底 | https://code.claude.com/docs/en/cross-session-messaging ｜已验证 | 部分并进：巡检专抓不结束的线。桌面端线结束一轮本来就会通知总控 |
| 另一家模型审（OpenAI 官方的 codex 插件，在 Claude Code 里 `/codex:review`） | https://github.com/openai/codex-plugin-cc ｜已验证 | 写在 `tools.md`，当可选；它自己也提醒很费额度 |
| 委派时任务书写：目标、输出格式、用什么工具和资料、边界；子 agent 把结果写成文件、只回传路径 | https://www.anthropic.com/engineering/multi-agent-research-system ｜已验证 | 和任务书模板、任务书放文件只给路径一致 |

## 和这套冲突的地方

| 说法 | 出处 | 这套怎么办 |
| --- | --- | --- |
| Agent teams（领头 + 队友，共享任务表）是官方的多 agent 协作 | https://code.claude.com/docs/en/agent-teams ｜已验证 | 不拿来改代码：实验功能，队友没有各自的工作副本、不能再开队友、`/resume` 恢复不了。并行查资料、并行审可以用 |
| 默认思考强度是 medium；改疑难 bug 调到 high / xhigh | https://code.claude.com/docs/en/model-config ｜已验证 | 按作者的用法（2026-11-16）：明确的任务中就够；讨论、设计方案才开高，最强档用高；极高基本不用，慢、费劲，还爱自己扩大范围 |
| 用 `/loop` 定时巡检会重发总控整段上下文，费额度 | 研究子 agent 的推断，没验证 | 巡检提示词写明「要轻」，总控上下文到八成先存档再压缩 |

## 社区

| 谁 | 说了什么 | 出处 |
| --- | --- | --- |
| Boris Cherny（Claude Code 负责人） | 终端同时跑 5 个、网页再跑 5–10 个；靠系统通知知道谁要他；给 Claude 一个自己验收的办法，质量能高 2–3 倍 | https://x.com/bcherny/status/2007179832300581177 ｜没验证（只看到 VentureBeat 转述） |
| Simon Willison | 并行的瓶颈在人审代码；适合研究、小维护、规格写清了的活 | https://simonwillison.net/2025/Oct/5/parallel-coding-agents/ ｜已验证 |
| ccusage | `blocks` 按 5 小时窗口统计，是从本地日志估的 | https://github.com/ryoppippi/ccusage ｜已验证 |
| Claude-Code-Usage-Monitor | 按消耗速度预测几点用完；它自己说状态栏 `rate_limits` 更准 | https://github.com/Maciek-roboblog/Claude-Code-Usage-Monitor ｜已验证 |
| ponytail（DietrichGebert/ponytail，MIT） | 让 agent 少写代码的一份 prompt：写之前走梯子（它原文七级，这里把标准库和平台并成一级，共六级，见 `line-rules.md`「写代码」），永远不省校验、防丢数据、安全、无障碍；有风险的逻辑留一个小测试；`shortcut:` 注释标出捷径的限制；审查「没有具体场景就不算问题」、每条写是什么 / 问题 / 修法 / 不修会怎样。作者自测（Opus 5.5，39 个任务各 5 次）：代码少 53%、花费少 26%，有风险的逻辑带测试 98%（不装 68%）；独立测试也说代码确实少，但必须写完大块功能的 agent 任务里，工具调用和花费可能反而变多 | https://github.com/DietrichGebert/ponytail ｜读了 skill 原文和 benchmark 说明（2026-11-18）；数字是对方自测的，没自己复现。并进了 `line-rules.md`「写代码」、报告的「梯子」、任务书「能复用的 / 不做」、`review.md`「先看」和「给审查员的话」、项目卡「预期负载」。没并的：ultra 档（会反过来质疑需求，和「只做任务书里的事」冲突）、状态栏和看板的钩子。装了 ponytail 插件也不冲突，线保持默认的 full 档 |
| Ralph 循环 | 每轮只做一件事，状态放在计划文件里，靠测试和类型检查挡坏输出 | https://ghuntley.com/ralph/ ｜已验证（发布日期没验证） |

## Codex 和腾讯

| 说法 | 出处 |
| --- | --- |
| AGENTS.md 逐层读、合计 32 KiB 上限 | https://learn.chatgpt.com/docs/agent-configuration/agents-md ｜已验证 |
| skill 放 `.agents/skills`、`~/.agents/skills`，不读 `.claude/skills` | https://learn.chatgpt.com/docs/build-skills ｜已验证 |
| 子 agent 只在明说时才开；`.codex/agents/*.toml` 可写模型和思考强度 | https://learn.chatgpt.com/docs/agent-configuration/subagents ｜已验证 |
| `--worktree`、`codex agents`、`codex exec`、`resume` / `fork` | https://learn.chatgpt.com/docs/developer-commands?surface=cli ｜已验证 |
| 额度：Plus、Business 有 5 小时额度，Pro 没有；`/status` 能不能看剩余额度两页说法不一 | https://learn.chatgpt.com/docs/pricing ｜没验证（冲突） |
| 定时只能在 ChatGPT 网页或桌面 app 里建 | https://learn.chatgpt.com/docs/automations ｜已验证 |
| WorkBuddy 支持 skill，路径 `~/.workbuddy/skills` | https://www.woshipm.com/ai/6424317.html ｜第三方文章；作者电脑上 WorkBuddy AI（agent-cli 2.165.0）程序里核到会扫 `~/.workbuddy/skills` 和项目 `.codebuddy/skills` |
| WorkBuddy 后台子 agent 是同一进程里的「队友」，不给工作副本、不能再开队友；agent 定义可写 model / effort / isolation；默认权限 bypass；计费工具在任何模式都弹窗；有 CronCreate | 作者电脑上的程序 `/Applications/WorkBuddy AI.app/…/cli/dist/codebuddy-lite-wb.mjs` 和内置钩子 `high-credit-gate.cjs` ｜读的是程序，没在 WorkBuddy 里实际跑过 |
| CodeBuddy Code：读 `CODEBUDDY.md` 或 `AGENTS.md`，skill 在 `.codebuddy/skills`，子 agent 最多嵌套 5 层 | https://www.workbuddy.ai/docs/cli/memory 、https://www.workbuddy.ai/docs/cli/sub-agents ｜已验证 |
