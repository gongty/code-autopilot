# code-autopilot

写代码、做项目的自动驾驶。用 Claude Code（或 Codex、WorkBuddy）做大项目：一个总控对话带 N 条线，全自动推进，又不撞额度。

- **总控当总工程师**：只定方案、写任务书、验收、合并。改代码的开成线（独立会话，各自的工作副本和分支），查找和审查交给子 agent。
- **按难度派模型和思考强度**：要判断边界的给强模型，任务书写到能照单执行的给便宜一档，最贵的只在你点名时用。
- **定时巡检**：有线在跑就每 20 分钟看一次，卡住、原地打转、超时的当场纠正；没线在跑就删掉。
- **额度管家**：每次巡检查 5 小时和周额度，推算到重置时会不会超；不够就让线停在一个提交上、存好进度，定时等额度恢复再接着做。

来历：2026-11-15 到 16，一个总控对话带 7 条线做完一次完整重构，夜里没人盯，5 小时额度没撞墙。真实读数和踩过的坑在 `references/quota.md`、`references/review.md`。

## 安装

整个仓库就是一个 skill，克隆进 skill 目录就装好了：

```bash
git clone https://github.com/gongty/code-autopilot ~/.claude/skills/code-autopilot
```

| 用什么 | 克隆到哪 |
| --- | --- |
| Claude Code（桌面端、命令行都行），所有项目通用 | `~/.claude/skills/code-autopilot` |
| Claude Code，只给一个项目用 | `<项目>/.claude/skills/code-autopilot` |
| Codex | `~/.agents/skills/code-autopilot`（Codex 不读 `.claude/skills`） |
| 腾讯 WorkBuddy | `~/.workbuddy/skills/code-autopilot`（还没真跑过，见 `references/workbuddy.md`） |

更新：在那个目录里 `git pull`。同一份 `SKILL.md` 几家都能读；工具叫法不同的地方写在 `references/tools.md`。WorkBuddy 的总控开不了新对话，线改成后台队友或你手动开的对话，写在 `references/workbuddy.md`。

## 用法

装好以后，对总控说「这个重构你当总控，全自动推进」之类的话就会用上。第一次在一个项目里用，它会先和你一起填项目卡 `.code-autopilot/project.md`（测试命令、门、端口、不许碰的东西）。

## 文件

| 文件 | 给谁看 | 写什么 |
| --- | --- | --- |
| `SKILL.md` | 总控 | 全部规矩，一页 |
| `references/line-rules.md` | 每条线 | 线要守的（复制成项目的 `.code-autopilot/briefs/common.md`） |
| `references/patrol.md` | 总控 | 巡检、停手、续上三段提示词 |
| `references/quota.md` | 总控 | 额度推算和一晚上的真实读数 |
| `references/review.md` | 总控 | 验收、审查、合并的细则 |
| `references/tools.md` | 总控 | 四样能力在 Claude Code 桌面端、命令行、Codex 里叫什么 |
| `references/workbuddy.md` | 总控 | 在 WorkBuddy 里没有子对话时怎么开线、巡检、管积分 |
| `references/sources.md` | 想深究的人 | 官方文档和社区做法的出处，和这套做法哪里不同 |
| `templates/` | 总控 | 项目卡、任务书、总控备忘的模板 |
