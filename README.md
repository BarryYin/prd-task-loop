# PRD Task Loop

Autonomous PRD-driven development loop. Reads a PRD/task list, loops through tasks using AI coding agents, auto-commits, and marks tasks complete.

Inspired by [Ralphy](https://github.com/michaelshimeles/ralphy), adapted for the [Hermes Agent](https://github.com/hermes-agent) skill ecosystem.

## Install

```bash
# Any agent (Claude Code, OpenCode, Codex, Cursor, etc.)
npx skills add BarryYin/prd-task-loop

# Global install (all projects)
npx skills add BarryYin/prd-task-loop -g
```

## What It Does

1. Read a PRD file (Markdown or YAML)
2. Find the next incomplete task (`- [ ]`)
3. Run an AI coding agent (OpenCode, Claude Code, Codex...)
4. Auto-commit changes
5. Mark task complete in the PRD (`- [x]`)
6. Repeat until all tasks done

## Usage

### Single Task

```bash
ralphy --opencode "add login button"
```

### PRD Mode (sequential)

Create a `PRD.md`:

```markdown
## Features
- [ ] Create user model
- [ ] Add auth endpoints
- [ ] Build dashboard
```

Then run:

```bash
ralphy --opencode --prd PRD.md
```

### Parallel Execution (YAML)

```yaml
tasks:
  - title: Create user model
    completed: false
    parallel_group: 1
  - title: Create post model
    completed: false
    parallel_group: 1  # runs in parallel with above
  - title: Add relationships
    completed: false    # runs after group 1
```

```bash
ralphy --opencode --yaml tasks.yaml --parallel
```

## Supported Engines

| Engine | Flag | Command |
|--------|------|---------|
| OpenCode | `--opencode` | `opencode run '<prompt>'` |
| Claude Code | `--claude` (default) | `claude -p '<prompt>'` |
| Codex | `--codex` | `codex exec --full-auto` |
| Cursor | `--cursor` | `agent --print --force` |
| Copilot | `--copilot` | `copilot -p '<prompt>'` |
| Qwen | `--qwen` | `qwen -p '<prompt>'` |

## With Hermes Agent

This skill integrates with Hermes Agent's tool ecosystem:

- `terminal()` — run AI coding agents
- `delegate_task()` — parallel task execution
- `patch()` — update PRD task status
- `session_search()` — recall past runs
- `memory` — persist project context

## Configuration (optional)

Create `.ralphy/config.yaml` in your project:

```yaml
project:
  name: "my-app"
  language: "TypeScript"
  framework: "Next.js"

commands:
  test: "npm test"
  lint: "npm run lint"

rules:
  - "use strict TypeScript"
  - "follow existing code patterns"

boundaries:
  never_touch:
    - "*.lock"
    - "src/legacy/**"
```

## Comparison with Ralphy

| | Ralphy (bash) | This Skill |
|---|---|---|
| Setup | `npm i -g ralphy-cli` | `npx skills add BarryYin/prd-task-loop` |
| Flexibility | Fixed bash logic | Full LLM + tool access |
| Debugging | Shell scripts | Inspect each step |
| Integration | Standalone | Part of agent ecosystem |

## License

MIT
