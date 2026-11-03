# agent-skills
Small, plain agent skills. Each is a folder with a `SKILL.md`, the open Agent Skills format, so it works in Claude (Claude Code and the Claude apps), OpenAI Codex, and other tools that read that format.

## Skills
| Skill | What it does |
|---|---|
| [`quota-burn`](quota-burn/SKILL.md) | Turns spare AI quota into useful, checked work. It asks how much quota you have and how fast to use it, you write the backlog in plain words, and it posts every checked result to a board (JSONL in git by default). |

## Install
- **Claude Code:** copy the folder into `~/.claude/skills/`.
- **Codex:** copy it into `~/.codex/skills/`.
- **Claude apps:** add the zipped folder as a skill in your Claude settings.

Then say "burn my quota" or "use quota-burn".
