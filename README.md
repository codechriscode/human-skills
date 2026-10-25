# agent-topics

Talk to subagents clutter-free until you reach a conclusion with them, then unite everything in the main session.
Great for keeping many things on track at once and dividing your focus with intent.

Leverages shells to keep subagent sessions alive.

## Skills

| Skill | What it does | Claude Code | opencode |
| --- | --- | --- | --- |
| [agent-topics](claude/agent-topics/skills/agent-topics/SKILL.md) | Split a discussion into topics, one subagent per topic. Chat with each agent directly; they idle between messages and report back only when told to wrap. | ✅ | 🚧 in progress |
| [walk-me-through](claude/walk-me-through/skills/walk-me-through/SKILL.md) | Teach a senior developer one subject (architecture, data, business rules, a ticket or a PR) in 3-5 short turns, with a quiz before each new turn and pickable teaching frameworks. | ✅ | — |

## Install — Claude Code

Add this repo as a plugin marketplace and install the skill:

```sh
claude plugin marketplace add codechriscode/agent-topics
claude plugin install agent-topics@agent-topics
claude plugin install walk-me-through@agent-topics
```

Or from inside a session:

```
/plugin marketplace add codechriscode/agent-topics
/plugin install agent-topics@agent-topics
```

The skill loads as `agent-topics` (prefixed `agent-topics:` in plugin context). Restart your session or run `/reload-plugins` if it doesn't show up right away.

### Manual install (no marketplace)

Copy the skill folder into your personal skills directory:

```sh
git clone https://github.com/codechriscode/agent-topics
mkdir -p ~/.claude/skills
cp -r agent-topics/claude/agent-topics/skills/agent-topics ~/.claude/skills/
```

## Install — opencode (in progress, being tested)

The opencode flavor ships two files, and opencode keeps skills and commands in separate directories, so both get copied:

```sh
git clone https://github.com/codechriscode/agent-topics
mkdir -p ~/.config/opencode/skills/agent-topics ~/.config/opencode/command
cp agent-topics/opencode/agent-topics/SKILL.md ~/.config/opencode/skills/agent-topics/
cp agent-topics/opencode/agent-topics/command/agent-topics.md ~/.config/opencode/command/
```

`/agent-topics <topics>` then works as a literal slash command, and the skill also triggers by phrase.

> Both flavors share the skill name. Install only the flavor for your harness: opencode also reads `~/.claude/skills/`, so having both flavors in directories it scans causes a name collision.

## For maintainers

After editing a plugin, validate before pushing:

```sh
claude plugin validate .
```

## License

MIT
