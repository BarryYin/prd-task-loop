# PRD Task Loop

Autonomous PRD-driven development loop. Reads a PRD/task list, loops through tasks using AI coding agents or Hermes subagents, auto-commits, and marks tasks complete.

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
2. Resolve task dependencies and execution order
3. Run AI agents/subagents (OpenCode, Claude Code, Codex, Hermes subagents)
4. Auto-commit changes
5. Mark task complete in the PRD
6. Repeat until all tasks done

## What's New in v2.0.0

- **Subagent execution** — use Hermes `delegate_task` for full tool access per task
- **3 parallel strategies** — subagent parallel, git worktree isolation, hybrid
- **Dependency graph** — topological sort with `depends` field and cycle detection
- **Dry-run mode** — preview execution plan before running
- **Config system** — `.prd-loop/config.yaml` for project/execution/verification settings
- **Structured progress** — JSON state + human-readable Markdown log
- **PRD auto-generation** — analyze project structure and generate tasks automatically
- **Skill composition** — pipelines with kiro-spec-tasks, writing-plans
- **Enhanced parsing** — `[priority]`, `[depends]`, `[tags]`, `[engine]` metadata in Markdown

## Usage

### Single Task

```bash
ralphy --opencode "add login button"
```

### PRD Mode (sequential)

Create a `PRD.md`:

```markdown
## Features
- [ ] Create user model [priority:high]
- [ ] Add auth endpoints [depends:user-model]
- [ ] Build dashboard [depends:auth-endpoints]
```

Then run:

```bash
ralphy --opencode --prd PRD.md
```

### YAML Tasks with Dependencies

```yaml
tasks:
  - id: setup-db
    title: Setup database schema
    priority: high

  - id: user-model
    title: Create user model
    depends: [setup-db]
    parallel_group: 1

  - id: post-model
    title: Create post model
    depends: [setup-db]
    parallel_group: 1  # runs in parallel with user-model

  - id: relationships
    title: Add model relationships
    depends: [user-model, post-model]  # runs after both
```

```bash
ralphy --opencode --yaml tasks.yaml --parallel
```

### Hermes Subagent Mode (Recommended)

In Hermes, this skill uses `delegate_task` for each task:

```python
# Sequential
result = delegate_task(
    goal="Create user model with email, password fields using Prisma",
    context="Project: Next.js + Prisma at ~/my-project",
    toolsets=["terminal", "file"]
)

# Parallel (multiple tasks at once)
results = delegate_task(tasks=[
    {"goal": "Create utils.js", "context": "...", "toolsets": ["terminal", "file"]},
    {"goal": "Create types.ts", "context": "...", "toolsets": ["terminal", "file"]},
])
```

## Execution Modes

| Mode | Best For | Tools |
|------|----------|-------|
| **Subagent** | Complex multi-file tasks, needs reasoning | Full (file, terminal, web, browser) |
| **Engine** | Simple, known patterns | Terminal only |
| **Hybrid** | Worktree isolation + subagent reasoning | Both |

## Parallel Strategies

| Strategy | When to Use |
|----------|-------------|
| **A: Subagent Parallel** | Complex tasks, need full reasoning |
| **B: Worktree Parallel** | Tasks modifying overlapping files |
| **C: Hybrid** | Complex tasks + file conflicts |

## Configuration

Create `.prd-loop/config.yaml`:

```yaml
project:
  name: "my-app"
  language: "TypeScript"
  framework: "Next.js"

execution:
  mode: "subagent"        # subagent | engine | hybrid
  parallel: true
  max_concurrent: 3
  auto_commit: true

verification:
  test_command: "npm test"
  run_tests: true

rules:
  - "use strict TypeScript"
  - "follow existing code patterns"

boundaries:
  never_touch:
    - "*.lock"
    - "src/legacy/**"
```

## Supported Engines

| Engine | Command |
|--------|---------|
| OpenCode | `opencode run '<prompt>' --format json` |
| Claude Code | `claude --dangerously-skip-permissions -p '<prompt>'` |
| Codex | `codex exec --full-auto --json '<prompt>'` |
| Subagent | `delegate_task(goal=...)` |

## With Hermes Agent

This skill integrates with Hermes Agent's tool ecosystem:

- `terminal()` — run AI coding agents
- `delegate_task()` — parallel task execution
- `patch()` — update PRD task status
- `session_search()` — recall past runs
- `memory` — persist project context

## Skill Composition

```
kiro-spec-tasks → prd-task-loop     # spec → auto-execute
writing-plans → prd-task-loop       # plan → auto-execute
```

## Comparison with Ralphy

| | Ralphy (bash) | Engine Mode | Subagent Mode |
|---|---|---|---|
| Setup | `npm i -g ralphy-cli` | No install | No install |
| Parallel | Git worktrees | Manual | Built-in |
| Tools | Terminal only | Terminal only | Full access |
| Reasoning | None | CLI agent | Full LLM |
| Cost | Lowest | Low | Higher |

## License

MIT
