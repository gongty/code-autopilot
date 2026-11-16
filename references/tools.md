# 四样能力在各家叫什么

code-autopilot 只要四样能力：**开线**（开一个带工作副本的独立会话）、**看线和发话**、**查额度**、**定时**。下表是各环境里的叫法，WorkBuddy 见 `workbuddy.md`。标「没验证」的是查文档得来、还没在真项目里跑过的。

## Claude Code 桌面端（Code 标签页）——这套做法就是在这里跑出来的

| 能力 | 工具 | 要点 |
| --- | --- | --- |
| 开线 | `start_session` | `use_worktree: true` 给它自己的工作副本和分支；`branch_name` 只是个短名，环境会加用户的分支前缀和后缀，真实分支名看返回结果；`source_branch` 指定从哪个分支开；`model` 选模型（只能同一家或更便宜，填更高的不报错、直接被忽略），`effort` 选思考强度（不高于总控）；`context: "fresh"` 时要给 `prompt`（任务）和 `background`（背景）。`initiation: "user_asked"` 只在用户亲口要过并行做、或叫你开这一块时用（开工对齐时用户答应了就算），不然只能提一张卡片等用户点 |
| 看线 | `list_sessions {"linked": true}`、`get_session` | 列出这个总控开的所有线：在跑 / 闲着 / 等你、分支、工作副本 |
| 读报告 | 读 `.code-autopilot/reports/<线名>.md` | 线结束一轮时总控收到的通知只带开头几百字；要看会话原文用 `list_events` |
| 发话 | `SendMessage`（或 `send_message`） | 线在等一个卡住的命令时，消息排着读不到；先停掉那个进程 |
| 停线 | `stop_session` | 打断这一轮（和用户点停止一样），会话还在，之后还能发话接着做。它在跑的前台命令会不会一起停：没验证，停完看一眼进程表 |
| 中途换模型 | `set_session_model`、`set_session_effort` | 只能改总控开的线，改不了总控自己。总控的模型、思考强度、权限模式由用户在会话里设；`set_session_permission_mode` 往上切（比如切到 auto）会弹卡片等用户批 |
| 查额度 | `get_usage` | 一次给全：5 小时窗口 %、周额度 %（全部模型、单个模型）、各自几点重置、超额付费开没开，还有这个会话的上下文用了多少 |
| 定时 | `CronCreate` / `CronList` / `CronDelete` | 本地时间的 5 段 cron；`recurring: false` 是只跑一次。只活在这个会话里，关掉应用就没了；循环的 7 天后自动过期；只在会话空着时触发（正忙就等忙完）；循环的会晚触发最多半个周期（最多 30 分钟），所以别卡着整点 |
| 防休眠 | `request_keep_awake` | 只防闲置休眠，合盖照样睡；会话空了约 5 分钟就放开。所有线都停着等重置的那几个小时盖不住，用 `caffeinate -i -t <秒数>` |
| 跨重启的定时 | 桌面端的「定时任务」（scheduled tasks），`fireAt` 只跑一次（一次性的别写 cron）、`cronExpression` 循环 | 写在磁盘上，应用开着才跑，关着时到点就在下次打开时补跑。**每次是一个全新的运行，不记得总控的对话**，提示词要自带全部信息；跑完会通知开它的会话。所以它叫不醒「带着上下文的总控」，只能做自带信息的事；能不能靠跑完的通知把总控叫起来：没验证 |
| 子 agent | `Agent` | `model` 单独选；`run_in_background` 后台跑，跑完通知；新版能往下再开（见命令行一节），但改代码、要往下派活的还是开成线 |
| 设计线 | `start_session` 开一条线，在里面用 `Artifact` 从 Design 类型新建画布出稿 | Claude Design 已并进 Claude；账号里要有 Design 类型（`Artifact` 列类型能看到）。稿子是一张设计画布，用户能看、能留言。Codex、WorkBuddy 没有，见 `design.md` 最后一节 |

桌面端的两个限制（实测）：

1. **只有用户亲口发话的那一轮能直接开线。** 由定时、别的会话的消息叫醒的那一轮，开线会被拦下，只能提一张卡等用户点。对策：能预见的线在用户发话那轮一次开好；后面的活发给做完、还没归档的线接着做。
2. **开到别的文件夹的线，也要用户在这个会话里说。** 总控只能在自己的项目里自动开线。

## Claude Code 命令行（查官方文档得来，2026-11-16，没在真项目里跑过）

`claude -w` 开的是交互会话，总控在自己的命令行里替线开不了；能自动开、自动跑的是后台会话 `claude --bg`。

| 能力 | 怎么做 | 要点 |
| --- | --- | --- |
| 开线 | `claude -w <线名>`（带工作副本）；或 `claude --bg` 开后台会话（研究预览，改代码前会自动挪进工作副本，关掉终端照跑） | 工作副本在 `.claude/worktrees/<线名>`，分支 `worktree-<线名>`；`.env` 这类被 gitignore 的文件要写进 `.worktreeinclude` 才会带过去；`--model`、`--effort` 选模型和思考强度 |
| 看线 | `claude agents`（`--json` 给每条的状态：working / blocked / done / failed） | 研究预览 |
| 发话 | 跨会话 `SendMessage`；`notify_when_idle: true` 让对方闲下来时回一次通知（12 小时内有效） | 别的会话发来的消息不能替用户批准权限 |
| 接着做 | `claude --resume` / `--continue` | 会回到原来的工作副本 |
| 查额度 | `/usage`；状态栏脚本收到的 JSON 里有 `rate_limits.five_hour` 和 `rate_limits.seven_day`（`used_percentage`、`resets_at`） | 状态栏这份只给 Pro / Max / Ultra，第一次回复以后才有。要让总控读到，可以让状态栏脚本顺手写进一个文件（自己的做法，没验证） |
| 定时 | `/loop`、`CronCreate` | 同桌面端：会话关了就没了，7 天过期，只在空着时触发；一个会话最多 50 个 |
| 跨重启的定时 | 云端定时（`/schedule`） | 最小间隔 1 小时，每次是新克隆的仓库，读不到本地文件，也占订阅额度 |
| 子 agent | `.claude/agents/*.md` 的 frontmatter 写 `model`、`effort` | 新版默认能往下开三层，`CLAUDE_CODE_MAX_SUBAGENT_SPAWN_DEPTH` 改（设 1 就不能嵌套） |
| 撞额度 | 交互会话默认在原地等，重置后自己接着做，最多连着 3 次 | 周额度、后台会话、`-p` 不会自动接 |

另有「agent teams」（实验功能，`CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1`）：一个领头的带几个队友，靠共享任务表和信箱协调。队友没有各自的工作副本、不能再开队友、恢复不了，**不适合拿来并行改代码**，适合并行查资料、并行审。

## Codex（查官方文档得来，2026-11-16，没在真项目里跑过）

| 能力 | 怎么做 | 要点 |
| --- | --- | --- |
| 读规矩 | `AGENTS.md` | 先读 `~/.codex/AGENTS.md`（或 `AGENTS.override.md`），再从仓库根往当前目录逐层读，越近越优先；加起来默认上限 32 KiB |
| 读 skill | `~/.agents/skills/code-autopilot/` 或仓库里 `.agents/skills/code-autopilot/` | SKILL.md 格式通用，但 **Codex 不读 `.claude/skills`** |
| 开线 | `codex --worktree`、`codex exec`（不交互地跑一件事）、`codex resume` / `fork` | |
| 看线 | `codex agents` | |
| 子 agent | `.codex/agents/*.toml`，可写 `model`、`model_reasoning_effort`；内置 default / worker / explorer | **只有你明说、或 AGENTS.md / skill 里写了才会开**，所以总控的提示里要写「查找和审查用子 agent」。能不能嵌套：没验证 |
| 选模型 | 复杂、要设计方案的用 Astra 6；简单、任务书写清楚的用 6.1 sol 完全够 | 作者的用法（2026-11-16） |
| 思考强度 | `model_reasoning_effort`，或 `/model --reasoning=<档>` | 明确的任务用中；讨论、设计方案用高；极高基本不用（慢、费劲，爱自己扩大范围） |
| 查额度 | 网页 chatgpt.com/codex/settings/usage；命令行 `/status` 或 `/usage`（官方两页说法不一，没验证） | Plus、Business 有 5 小时额度，Pro 目前没有；可能还有周额度 |
| 定时 | 命令行没有；只能在 ChatGPT 网页或桌面 app 里建自动任务 | 巡检和叫醒得靠外面：比如系统定时（launchd、cron）到点跑一次 `codex exec "照 code-autopilot 巡检"`（没验证） |

用 Codex 当总控时，执行的线也用 Codex，别跨家派活；但审查可以借另一家（比如在 Claude Code 里装官方的 codex 插件，让 Codex 审 Claude 写的），一个写一个审，互相不护短。

## 上下文：自动压缩开着，窗口 400k

作者实测 400k 是甜点区：速度快，token 也不多。用户级 `~/.claude/settings.json` 里按模型设，总控和线用到的模型都要写上（改用户的设置，先问用户）：

```json
"modelSettings": {
  "claude-opus-5-5":   { "autoCompactWindow": 400000 },
  "claude-sonnet-5-5": { "autoCompactWindow": 400000 }
}
```

- 桌面端 `get_usage` 会报这个会话的上下文：用了多少、窗口多大、到几成自动压缩（实测 400k 窗口在 92% 时压）。
- Claude Code 程序里还有环境变量 `CLAUDE_CODE_AUTO_COMPACT_WINDOW`，用法没核。
- Codex：config.toml 里应有类似的自动压缩上限（`model_auto_compact_token_limit`），没验证。
- 压缩会丢细节，所以总控每次巡检先读 `notes.md`，到七八成先存档再压（2026-11-16 08:42 总控在 81% 时就是这么做的）。

## 权限：让线不停下来等人（查于 2026-11-16）

线一弹出权限请求，就只能等人点；别的会话发来的消息不算用户同意，总控批不了。所以要在开线之前就放开。

**Claude Code**

| 做法 | 怎么设 | 要点 |
| --- | --- | --- |
| 自动模式（首选） | 桌面端：请用户把总控切到 auto（总控自己往上切会弹卡片等用户批），再开线（`start_session` 的 `permission_mode` 不写就跟总控一样，不能比总控高）。命令行：`claude --permission-mode auto`；想默认就用它，`permissions.defaultMode: "auto"` 只能写在 `~/.claude/settings.json`，写在项目里不生效 | 一个单独的分类器在执行前审：正常的放行，越界的拒掉并把原因交回。**会退回逐条问人的情况**：连拒 3 次或累计拒 20 次；碰上 ask 规则。2026-11-15 到 16 总控和 7 条线全程 auto，被拒 2 次（删大目录的安全检查），线换了办法接着做，没停 |
| 白名单 / 黑名单 | `permissions.allow` / `deny` / `ask`，写在 `~/.claude/settings.json`（所有项目）或项目的 `.claude/settings.json`（进 git）、`.claude/settings.local.json`（不进 git）。改之前先问用户 | 先看 deny，再 ask，再 allow；几个文件里的规则合并。推主分支别写进 deny：总控和线读的是同一份，总控合完也推不上去。Bash 按前缀写：`Bash(npm test *)`。工作副本里的会话读的是主目录那份 `settings.local.json`；在工作副本里点「不再询问」也写回主目录，所有工作副本都生效。deny 在任何模式下都管用 |
| 没人应答就直接拒 | `--permission-prompts none`（2.1.259 起） | 只对 `--print`（`-p`）生效：要批准的调用直接拒掉，并告诉 Claude 别重试 |
| 沙箱 | `/sandbox`，或设置里 `sandbox.enabled: true` | 沙箱里的命令在自动放行的设定下不用逐条批。但只能写工作目录和临时目录，网络默认不通：要装依赖、要联网的命令会弹窗或失败；工作副本里提交要写主目录的 `.git`，会不会被拦：没验证。开之前想清楚 |
| 钩子自动批 | `PermissionRequest` 钩子返回 `"behavior": "allow"` | matcher 要收窄到具体命令；留空或 `.*` 等于什么都放 |
| 全放开 | `bypassPermissions` / `--dangerously-skip-permissions` | 不建议：没人看着时出错代价最大。deny 规则仍然管用 |

后台会话（`claude --bg`）遇到要批准的，会显示「Needs input」一直等。`claude -p` 默认把要批准的直接拒掉。

出处：code.claude.com/docs/en/permission-modes、/permissions、/settings、/worktrees、/sandboxing、/hooks、/headless、/agent-view、/cross-session-messaging。

**Codex**

| 做法 | 怎么设 | 要点 |
| --- | --- | --- |
| 不问人、只在工作目录里写 | `--ask-for-approval never --sandbox workspace-write` | 不问人，但仍在沙箱里；网络默认关 |
| `codex exec` 跑一条线 | 要显式加 `--sandbox workspace-write` | 默认是只读沙箱，写不了代码 |
| 全放开 | `danger-full-access` | 官方：只在隔离的机器或容器里用 |

`--full-auto` 已经不支持了。出处：learn.chatgpt.com/docs/agent-approvals-security、/non-interactive-mode。

## 腾讯 WorkBuddy

总控开不了新对话，四样能力缺第一样。替代办法单独写在 `workbuddy.md`：小线开成后台队友（自动），大线由你手动开新对话（三层结构不变）。
