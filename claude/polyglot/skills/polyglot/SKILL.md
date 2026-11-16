---
name: polyglot
description: Talk to the user in their chat language and write to each other audience (Jira, Slack, GitHub, Confluence, email, code comments, commits) in the language that audience uses. The rules load in every session from a short block in the user CLAUDE.md, so the skill costs nothing until a new environment appears. Use when the user says "/polyglot", "set my languages", "speak <language> to me", "write Jira in English", or when you are about to write for an environment that has no language rule yet.
---

# Polyglot

The user chats in one language. Each **environment** (a place where other people read what you write) has its own language. For example, the user chats in Portuguese, Jira is in English, and one Slack channel is in Spanish. You write each text in the language of the people who read it.

## The rules block

The rules live in a block in `~/.claude/CLAUDE.md`, between two marker lines. Claude Code loads that file in every session, so the rules are always in context. You do not load this skill to follow them. You load it only to set up the rules or to add one.

```
<!-- polyglot:start -->
## Languages (polyglot)
- Chat with me in: Portuguese
- Everything else: English
- Rules, most specific first. A text for an environment with no rule: load the polyglot skill.
  - slack:#time-br — Portuguese (detected 2026-11-16)
  - jira — English (user)
<!-- polyglot:end -->
```

- **Chat** covers every answer to the user in the conversation, and the questions and summaries you give the user.
- **Everything else** is the default for every environment that has no rule. It is optional. If it is set, you need no detection. If it is not set, you detect the language once per new environment.
- **Environment keys** go from general to specific: `jira`, `jira:MAX`, `slack`, `slack:#channel`, `github`, `github:owner/repo`, `confluence:SPACE`, `email:domain.com`, `code:owner/repo` (code comments, commit messages, test names, docs in the repo). The most specific key that matches wins.
- **Source** is `user` (the user said it) or `detected <date>`. A `user` rule always wins over a detection. Never change a `user` rule without the user.

## Setup (`/polyglot`, or the first use with no block)

1. Ask at most two questions in one AskUserQuestion call: "Which language do we chat in?" and "Which language for everything else (Jira, Slack, GitHub, code)?". Offer "Detect per environment" as an option for the second question. Skip a question if the user already answered it in the message, for example `/polyglot chat pt, rest en`.
2. If the user names environment rules in the same message, add them with the source `user`.
3. Write the block into `~/.claude/CLAUDE.md`. Replace the old block if one exists. Append it at the end if none exists. Do not change anything outside the markers.
4. Confirm in one line, in the chat language.

To change a rule later, the user says it in plain words ("Slack #ops is in Spanish"). Update the block and confirm in one line.

## Writing for an environment

Before you write a text that other people read, find its environment key and its rule.

1. **A rule matches.** Use it. Do no fetch.
2. **No rule, and "Everything else" is set.** Use it. Do no fetch. But if text from that environment is already in your context for this task (the ticket you read, the thread you reply to, the PR you review) and it is clearly in another language, ask the user one question: "`<key>` looks like `<language>`. Write in `<language>` and remember it?" Record the answer with the source `user`.
3. **No rule, and no "Everything else".** Detect the language:
   - First use text already in your context for this task. Look only at text that people wrote (descriptions, comments, messages, commit subjects). Ignore code, logs, field names, and bot text.
   - Only if nothing is in context, make one small read of that environment: the last 5 to 10 human messages or comments, or `git log -10 --format=%s` for a repo. Never make a fetch only to detect a language when a fetch for the task itself returns the text you need.
   - If the text is mixed or too short, use the language of the most recent human messages. If it is still not clear, ask the user once.
4. **Record a new rule.** After a detection or an answer, add the rule to the block with its source. Use the key at the level where the language really applies: if every Jira project you saw uses English, record `jira`, not `jira:MAX`. Tell the user in one line in the chat language, for example "Rule added: `slack:#ops` — Spanish". This is how the skill learns: the next session loads the rule with the block and needs no detection.

## Writing rules

- Write the full text in the target language. Do not mix languages in one text, except for names, code, field names, and quotes.
- Keep every style rule that the user gave (tone, length, simplified English, no emojis). These rules apply in the target language too.
- Quote other people in their original language. In the chat, add a translation in the chat language after the quote if the languages differ.
- When you show the user a draft before you post it, show it in the target language, as it will appear. Add a one-line summary in the chat language only if the target language is not the chat language.
- File paths, identifiers, commands, and code stay as they are.

## Limits

- Do not translate text that the user wrote and asked you to post as-is.
- Do not detect a language for one-off private notes to the user (scratch files, local reports). They use the chat language unless the user says otherwise.
