# code-autopilot

[中文](README.md)

Autopilot for writing code and building projects. Use Claude Code (or Codex, WorkBuddy) for complex development: building a new project from zero, large features, refactors. One controller conversation leads N lines and drives the work end to end, without hitting usage limits.

The skill itself is written in Chinese; the agent reads it either way and answers in your language.

- **The controller is a top-tier architect and the owner of the project.** It is responsible for the whole result and aims for the best outcome at the lowest token cost, not just "done". It decides the plan and the order of work, writes briefs, reviews and merges. Code changes go to lines (separate sessions, each with its own worktree and branch); searching and reviewing go to subagents.
- **One kickoff message.** Everything you need to decide (who decides the plan, when it is due, how design is delivered, which actions need your approval) is asked once, at the start, with a recommendation for each. Reply "all as recommended" and it starts; no more scattered questions afterwards.
- **Development principles.** Design the structure from first principles. Generalize from one problem to the class it belongs to and fix the source, no patches. Less code and simpler is better. Reuse whatever can be reused, but only popular, minimal open-source projects. Nothing starts before what it depends on is settled, so no feature is built twice. Test in proportion: quick, targeted tests while developing, full testing at merge and cutover points. Docs and memory move with the code.
- **Design line.** For anything with a UI, a line produces the design with Claude Design and stores it in the repo. UI code uses a single set of design tokens; each delivery comes with screenshots that are checked against the design before merging, so what gets built matches the design. Ship directly, pick a direction first, or review every batch: you choose.
- **Model and effort per task.** Strong models for work that needs judgment; a cheaper tier for briefs written to be followed step by step. Fable is expensive and off by default; it asks you first.
- **Patrols.** While lines are running, it checks every 20 minutes and corrects lines that are stuck, looping, or over time.
- **Usage manager.** On by default, no approval needed. Every patrol reads the 5-hour and weekly usage and projects it to the next reset. If it won't fit, lines stop at a commit, a timer waits for the reset, and work resumes. Only an explicit "ignore usage, go as fast as possible" turns it off.
- **Fast merge gate.** Run only the affected tests; hard time limit per test file; lines and the controller run the same gate; one gate at a time; the same tree is never gated twice.

Origin: on 2026-11-15/16 the author used one controller conversation to lead 7 lines through a complete refactor. During the night of the 16th, 5 lines ran unattended without hitting the 5-hour limit. (The day before, the org's spend limit was hit and work sat idle for 3 hours, which is where the usage manager came from.) Real readings and lessons are in `references/quota.md` and `references/review.md`.

## Requirements

- **Claude Code desktop (Code tab) is the primary environment.** Starting lines, watching them, reading usage and the design line all use desktop tools; this workflow was developed there. The CLI and Codex instructions come from the official docs and have not been run on a real project; WorkBuddy was only checked by reading its program, not run.
- Set the controller to Opus 5.5, effort xhigh, permission mode auto, yourself in the session: lines cannot use a stronger model or higher effort than the controller, and without auto permissions lines stop and wait for you to click "allow".
- Subscription usage: by default 2–5 lines run in parallel, which burns usage roughly 2–5 times as fast. The usage manager keeps you off the wall, but using 10–20% of the weekly limit in one night is normal.

## Install

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

Update with `git pull` in that folder. The same `SKILL.md` works everywhere; where tool names differ, see `references/tools.md`. WorkBuddy's controller cannot open new conversations, so lines become background teammates or conversations you open by hand (`references/workbuddy.md`).

## Usage

Once installed, it is the default for any fairly complex development request: a new project from zero, a feature spanning several modules, a refactor. Small changes to one or two files are done directly as usual. You can also just say "be the controller for this project and drive it end to end".

- **Kickoff**: it sends one alignment message; reply "all as recommended" or change a few items. If you said "I'm going to sleep, keep going", it decides by its recommendations without waiting.
- **New project**: a skeleton line first sets up the structure, tests and the merge gate. Feature lines are opened and wait; they start once the skeleton is merged.
- **Refactor**: scope, handover from old to new features, and the development plan come first (if you chose to decide the plan together, lines start after you approve). If the project feels messy or the root cause is unclear, run a project audit first (`templates/audit-prompt.md`: read-only, produces a report).
- **With a UI**: a design line runs alongside, using Claude Design (Claude Code only).
- **First use in a project**: it creates a project card `.code-autopilot/project.md` (test commands, gate, ports, what not to touch, what you decided).

## Files

| File | For | Contents |
| --- | --- | --- |
| `SKILL.md` | controller | All the rules, one page |
| `references/line-rules.md` | every line | Rules for lines (copied to the project's `.code-autopilot/briefs/common.md`) |
| `references/patrol.md` | controller | Four ready prompts: patrol, pause, resume, correct |
| `references/quota.md` | controller | Usage projection and one night of real readings |
| `references/review.md` | controller | Review, merge, and how to speed up the merge gate |
| `references/design.md` | controller, design line | Design brief, three delivery modes, keeping the build matched to the design |
| `references/tools.md` | controller | The four capabilities in Claude Code desktop, CLI and Codex; permissions |
| `references/workbuddy.md` | controller | Running lines, patrols and credits in WorkBuddy without sub-conversations |
| `references/sources.md` | the curious | Official docs and community practice, and where this workflow differs |
| `templates/kickoff.md` | controller | The kickoff message |
| `templates/plan.md` | controller | Plan: reuse, steps and dependencies, schedule |
| `templates/project.md`, `brief.md`, `notes.md`, `design-brief.md` | controller | Project card, line brief, controller notes, design brief |
| `templates/audit-prompt.md` | read-only line | Project audit prompt: finds root causes of structural problems from code evidence, so the project does not turn into a mess |

## License

MIT, see `LICENSE`.
