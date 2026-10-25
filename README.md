# human-skills

The anti-meat-proxy skill set. Everything you need to be productive, continue to learn things yourself, and stay in the loop.

## Skills

| Skill | What it does | Claude Code | opencode |
| --- | --- | --- | --- |
| [agent-topics](claude/agent-topics/skills/agent-topics/SKILL.md) | Keep conversations manageable and profound. Go as far as you want to go on each topic until you have a precise decision. Split a discussion into topics, one subagent per topic. Chat with each agent directly; they idle between messages and report back only when told to wrap. | ✅ | 🚧 in progress |
| [walk-me-through](claude/walk-me-through/skills/walk-me-through/SKILL.md) | Teach a senior developer one subject (architecture, data, business rules, a ticket or a PR) in 3-5 short turns, with a quiz before each new turn and pickable teaching frameworks. | ✅ | 🚧 in progress |

## Install — Claude Code

Add this repo as a plugin marketplace and install the skill:

```sh
claude plugin marketplace add codechriscode/human-skills
claude plugin install agent-topics@human-skills
claude plugin install walk-me-through@human-skills
```

Or from inside a session:

```
/plugin marketplace add codechriscode/human-skills
/plugin install agent-topics@human-skills
```

The skill loads as `agent-topics` (prefixed `agent-topics:` in plugin context). Restart your session or run `/reload-plugins` if it doesn't show up right away.

### Manual install (no marketplace)

Copy the skill folder into your personal skills directory:

```sh
git clone https://github.com/codechriscode/human-skills
mkdir -p ~/.claude/skills
cp -r agent-topics/claude/agent-topics/skills/agent-topics ~/.claude/skills/
```

## Install — opencode (in progress, being tested)

The opencode flavor ships two files per skill, and opencode keeps skills and commands in separate directories, so both get copied:

```sh
git clone https://github.com/codechriscode/human-skills
for skill in agent-topics walk-me-through; do
  mkdir -p ~/.config/opencode/skills/$skill ~/.config/opencode/command
  cp human-skills/opencode/$skill/SKILL.md ~/.config/opencode/skills/$skill/
  cp human-skills/opencode/$skill/command/$skill.md ~/.config/opencode/command/
done
```

`/agent-topics <topics>` and `/walk-me-through <subject>` then work as literal slash commands, and both skills also trigger by phrase.

> Both flavors share the skill name. Install only the flavor for your harness: opencode also reads `~/.claude/skills/`, so having both flavors in directories it scans causes a name collision.

## For maintainers

After editing a plugin, validate before pushing:

```sh
claude plugin validate .
```

## License

MIT
