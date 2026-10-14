# /trouble — four-minds debug protocol for stuck incidents

<p align="center">
  <img src="trouble-skill.jpg" alt="The four minds of /trouble: Data, Sherlock Holmes, Linus Torvalds, and Brian Cox around a holographic debug console" width="720">
</p>

**Stop guessing. Make the agent argue until the evidence wins.**

`/trouble` is an open-source **AI agent skill** for hard debugging: production
incidents, debug loops, and “nothing I try is fixing this” bugs. The protocol
the model executes is YAML in `plugins/trouble/skills/trouble/SKILL.md` — ids
defined once, no repeated prose.

It installs on **Hermes Agent**, **Claude**, **Claude Code**, **GitHub Copilot
CLI**, **Grok**, **OpenAI Codex**, **DeepSeek Harness**, **Cline**, and
**OpenCode**. **NVIDIA NeMo Agent Toolkit** is a workflow framework, not a
skill host: point its agent prompt at that same file.

Four personalities — **Data** (Star Trek), **Sherlock Holmes**, **Linus
Torvalds**, and **Brian Cox** — collect verbatim evidence, rank hypotheses
with explicit **Bayesian** arithmetic, and **will not apply a fix until they
unanimously agree**. Every run ends with a machine-parseable
`TROUBLE_VERDICT_BEGIN` / `TROUBLE_VERDICT_END` block that pipelines can gate
on.

If your coding agent patches first and thinks later, this is the protocol that
slows it down on purpose.

## Why people install it

- **Root-cause analysis, not vibes.** Claims without a quoted log line, file,
  or command output are tagged unverified and cannot support a fix.
- **Incident response for agents.** Exit-code triage, MCP documentation
  survey, smuggled-assumption hunt, timeline / causality check.
- **A real agreement gate.** Data, Sherlock, Linus, and Brian Cox each vote
  yes/no with a 0–10 confidence score before any code change.
- **Host coverage.** One plugin leaf, GitHub `owner/repo` install — the same
  shape as marketplace skills such as
  [write-legible-c](https://github.com/7etsuo/write-legible-c).

## Install

The skill directory is `plugins/trouble/skills/trouble/` (`SKILL.md` inside).
Marketplace hosts can skip the clone. Copy hosts need it:

```bash
git clone https://github.com/coldcanuk/trouble.git
```

Below, `$SKILL` means `trouble/plugins/trouble/skills/trouble` from that clone.
Restart the agent session after a copy. A session that was already open will
not see a skill added underneath it.

`npx skills add` only finds this repo's nested `SKILL.md` with `--full-depth`.
That was checked with `npx skills add coldcanuk/trouble --list --full-depth`.

### Grok

```bash
grok plugin marketplace add coldcanuk/trouble
grok plugin install trouble --trust
```

Enable it if it shows as installed but inactive:

```bash
grok plugin enable trouble
```

Or in the TUI: `/plugins` → Plugins tab → select **trouble** → `Space` to
enable → `r` to reload (or start a new session).

One-shot from the plugin path:

```bash
grok plugin install coldcanuk/trouble#plugins/trouble --trust
```

Local clone: `grok plugin marketplace add ./trouble`, then the install above.

Skill-dir alternative (not the marketplace plugin): `npx skills add
coldcanuk/trouble --skill trouble --full-depth -a grok -g`. That writes
`~/.grok/skills/trouble/`. The marketplace install is the one this repo tests.

### Claude Code

```bash
claude plugin marketplace add coldcanuk/trouble
claude plugin install trouble@trouble
```

In a session: `/plugin marketplace add coldcanuk/trouble` then
`/plugin install trouble@trouble`.

Copy, if you do not want the marketplace:

```bash
mkdir -p ~/.claude/skills
cp -R "$SKILL" ~/.claude/skills/trouble
```

Project-local: `.claude/skills/trouble/`. The same command with
`-a claude-code -g` from `npx skills add coldcanuk/trouble --skill trouble
--full-depth` writes the global directory.

### Claude

Claude (claude.ai), not Claude Code, has no git marketplace. Upload a zip of
the skill folder.

1. Enable **Code execution and file creation** (Skills stay off without it).
2. Zip the `trouble` directory so the folder name matches the skill name:

```bash
cd trouble/plugins/trouble/skills
zip -r trouble.zip trouble
```

3. Settings → Capabilities → Skills → **Upload skill**, and upload
   `trouble.zip`.
4. Toggle the skill on. Custom uploads are private to that account.

Organization owners can provision the same zip for everyone: Organization
settings → Plugins & skills → Add → Upload a skill. Those also load in Claude
Code for users who sign in with that Claude account, unless
`syncClaudeAiSkills` is `false` in Claude Code managed settings.

### GitHub Copilot CLI

```bash
copilot plugin marketplace add coldcanuk/trouble
copilot plugin install trouble@trouble
```

In a session: `/plugin marketplace add coldcanuk/trouble`. Copilot reads
`.github/plugin/marketplace.json` (and also accepts `.claude-plugin/`).

Skill-dir alternative:

```bash
npx skills add coldcanuk/trouble --skill trouble --full-depth -a github-copilot -g
```

That writes `~/.copilot/skills/trouble/`. Project scope (omit `-g`) writes
`.agents/skills/trouble/`.

### Codex / `.agents`

```bash
codex plugin marketplace add coldcanuk/trouble
```

Codex reads `.agents/plugins/marketplace.json`. The plugin manifest is
`plugins/trouble/.codex-plugin/plugin.json`. After adding the marketplace,
install **trouble** from the plugin directory.

Copy: `~/.codex/skills/trouble/` or, for the skills CLI, `-a codex` (project
`.agents/skills/`, global `~/.agents/skills/`).

### Hermes Agent

Hub install (identifier is `owner/repo/path-to-skill-dir`; `hermes skills
inspect` resolves this path):

```bash
hermes skills install coldcanuk/trouble/plugins/trouble/skills/trouble --category software-development --yes
```

Slash command is `/trouble`. In the desktop app, run `/reload-skills` or
restart — `hermes skills list` updates immediately, the slash-command map does
not. Windows home is `%LOCALAPPDATA%\hermes`.

Copy:

```bash
mkdir -p ~/.hermes/skills/software-development
cp -R "$SKILL" ~/.hermes/skills/software-development/trouble
```

`npx skills add coldcanuk/trouble --skill trouble --full-depth -a hermes-agent -g`
writes `~/.hermes/skills/trouble/` (or `$HERMES_HOME/skills/trouble/`). Hermes
walks that tree, so either layout loads.

### DeepSeek Harness

DeepSeek Harness has **no** git marketplace catalog analogous to
`.grok-plugin/marketplace.json` or `.claude-plugin/marketplace.json`. Do not
look for `.deepseek-plugin/marketplace.json` — that file is not a DSH
interface. DSH discovers `SKILL.md` by copying the skill bundle into a skill
root (or via `npx skills add` when you want a generic Agent Skills installer).

The skills CLI has no DeepSeek agent. Copy the shipped skill directory:

```bash
mkdir -p ~/.dsh/skills ~/.agents/skills
cp -R "$SKILL" ~/.dsh/skills/trouble
cp -R "$SKILL" ~/.agents/skills/trouble
```

Project-local (highest precedence is `.dsh/skills`):

```bash
mkdir -p .dsh/skills .agents/skills
cp -R "$SKILL" .dsh/skills/trouble
```

Optional Agent Skills CLI (not a DSH marketplace):

```bash
npx skills add coldcanuk/trouble --skill trouble --full-depth
```

If that installer does not find the nested `SKILL.md`, use the copy commands
above. Restart the DSH session and invoke `/trouble`.

### Cline

Cline reads `.cline/skills/` and `~/.cline/skills/` (Windows:
`C:\Users\USERNAME\.cline\skills\`), plus `.clinerules/skills/` and
`.claude/skills/`. It does not document `.agents/skills/`, which is where
`npx skills add -a cline` writes, so copy:

```bash
mkdir -p ~/.cline/skills
cp -R "$SKILL" ~/.cline/skills/trouble
```

Project-local, and the path to commit for a team: `.cline/skills/trouble/`.
Invoke `/trouble`, or let the description match. Skills load at startup.

### OpenCode

OpenCode reads `.opencode/skills/<name>/SKILL.md`,
`~/.config/opencode/skills/<name>/SKILL.md`, and the Claude and `.agents`
skill directories. `metadata.opencode/slash` is `"true"` in this skill, so
`/trouble` is a TUI command.

```bash
npx skills add coldcanuk/trouble --skill trouble --full-depth -a opencode -g
```

That writes `~/.config/opencode/skills/trouble/`. Copy equivalent:

```bash
mkdir -p ~/.config/opencode/skills
cp -R "$SKILL" ~/.config/opencode/skills/trouble
```

Project-local: `.opencode/skills/trouble/`. Start a new session after install.

### NeMo Agent Toolkit

NeMo does not load `SKILL.md`. It is a workflow framework. Any string field
accepts `file://`, resolved from the workflow file's directory. Allowed
extensions include `.md` (also `.txt`, `.prompt`, `.j2`, `.jinja`, `.tpl`,
`.template`).

`tool_calling_agent` can take the protocol as its system prompt:

```yaml
workflow:
  _type: tool_calling_agent
  llm_name: nim_llm
  tool_names: []  # your tools
  system_prompt: file://plugins/trouble/skills/trouble/SKILL.md
```

`react_agent` is different. In nvidia-nat-langchain 1.8.0, a set
`system_prompt` replaces the default prompt, and the joined
`system_prompt` + `additional_instructions` must still contain `{tools}` and
`{tool_names}` or `nat` raises `Invalid system_prompt`. Leave `system_prompt`
unset and attach the protocol here:

```yaml
workflow:
  _type: react_agent
  llm_name: nim_llm
  tool_names: []  # your tools
  additional_instructions: file://plugins/trouble/skills/trouble/SKILL.md
```

Adjust the `file://` path so it is relative to the workflow file, or use an
absolute path. There is no NeMo plugin package in this repo: the protocol has
no engine to wrap.

## Use

- Slash command: `/trouble`
- Skills menu: `/skills trouble`
- Triggers: “troubleshoot”, “debug this”, “root cause”, “incident response”,
  “let's Bayesian this”, “bring in Sherlock/Linus/Data/Brian”, “we need the
  four minds”, or a problem that has already been worked on unsuccessfully

The agent must announce **Using the four-minds debug protocol** and follow
`order` in `SKILL.md`.

## What you get

```
plugins/trouble/
  plugin.json
  .claude-plugin/plugin.json
  .codex-plugin/plugin.json
  skills/trouble/
    SKILL.md          # YAML protocol the model executes
```

Host catalogs (all point at `./plugins/trouble`):

| Host | Catalog |
| --- | --- |
| Grok | `.grok-plugin/marketplace.json` |
| Claude Code | `.claude-plugin/marketplace.json` |
| Copilot CLI | `.github/plugin/marketplace.json` |
| Codex / `.agents` | `.agents/plugins/marketplace.json` |

Skill-directory hosts (no catalog in this repo): Hermes Agent, Claude,
DeepSeek Harness, Cline, OpenCode. NeMo uses `file://`, not a skill directory.

## Uninstall

```bash
grok plugin uninstall trouble --confirm
grok plugin marketplace remove coldcanuk/trouble

claude plugin uninstall trouble@trouble
# then remove the marketplace in /plugin

copilot plugin uninstall trouble
copilot plugin marketplace remove trouble --force

hermes skills uninstall trouble
```

For a copy install, delete the skill directory you created (`trouble/` under
that host's skills path). Claude uploads are removed in Settings → Skills.

## Validate (maintainers)

```bash
grok plugin validate ./plugins/trouble
python3 -m unittest discover -s tests -v
```

## License

[GNU General Public License v3.0 or later](LICENSE) (GPLv3).

Copyright (C) 2026 coldcanuk.

This program is free software: you can redistribute it and/or modify it under
the terms of the GNU General Public License as published by the Free Software
Foundation, either version 3 of the License, or (at your option) any later
version.
