# entirety

Cross-agent skills that help coding agents use [Entire](https://github.com/entireio/cli) context — Checkpoints, sessions, and git history.

## Why This Exists

The [Entire CLI](https://github.com/entireio/cli) captures the context behind code changes: prompts, transcripts, [Checkpoints](https://docs.entire.io/cli/checkpoints), and the decisions that led to each change. This repository packages agent-invokable workflows that teach coding agents how to use that context across development environments.

## Install

Install every skill with the [skills](https://skills.sh/) CLI:

```bash
npx skills add https://github.com/ronaldtebrake/entirety --all
```

Install one skill:

```bash
npx skills add https://github.com/ronaldtebrake/entirety --skill example
```

See [Agent-Specific Installation](#agent-specific-installation) for setup by agent.

## Quick Start

> [!NOTE]
> These skills are only useful in codebases with real [Checkpoints](https://docs.entire.io/cli/checkpoints) and session history.

After installing, ask your agent for a workflow:

```text
deslop this branch
```

## Included skills

Each skill lives in `skills/<skill-name>/SKILL.md`.

| Skill | Description |
| --- | --- |
| `deslop` | Remove AI slop from branch changes using checkpoint intent |

## Requirements

- the [Entire CLI](https://github.com/entireio/cli) installed
- a git repository with Entire sessions or Checkpoints
- [`entire login`](https://docs.entire.io/cli/commands#login) for workflows that search indexed history

## Agent-Specific Installation

<details>
<summary>Cursor</summary>

```bash
git clone https://github.com/ronaldtebrake/entirety.git ~/.cursor/skills/entirety
```

Cursor auto-discovers skills from `.agents/skills/` and `~/.cursor/skills/`.

</details>

<details>
<summary>Codex (OpenAI)</summary>

```bash
git clone https://github.com/ronaldtebrake/entirety.git ~/.agents/skills/entirety
```

Codex auto-discovers skills from `~/.agents/skills/` and `.agents/skills/`.

</details>

<details>
<summary>Claude Code</summary>

```bash
/plugin marketplace add ronaldtebrake/entirety
/plugin install entirety
```

</details>

<details>
<summary>Copilot</summary>

```bash
/plugin install https://github.com/ronaldtebrake/entirety
# or
git clone https://github.com/ronaldtebrake/entirety.git ~/.copilot/skills/entirety
```

</details>

<details>
<summary>Gemini CLI</summary>

```bash
gemini extensions install https://github.com/ronaldtebrake/entirety
```

</details>

<details>
<summary>OpenCode</summary>

```bash
git clone https://github.com/ronaldtebrake/entirety.git ~/.agents/skills/entirety
```

OpenCode auto-discovers skills from `.agents/skills/`, `.opencode/skills/`, and `.claude/skills/`.

</details>

## Contributing

See [CONTRIBUTING.md](CONTRIBUTING.md) for adding skills and validation steps.

## License

MIT — see [LICENSE](LICENSE).
