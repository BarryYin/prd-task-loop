---
name: prd-task-loop
description: "Autonomous PRD-driven development loop inspired by Ralphy. Reads a PRD/task list, loops through tasks using AI coding agents (OpenCode, Claude Code, Codex), auto-commits, and marks tasks complete. Supports parallel execution with git worktree isolation."
version: 1.0.0
author: Hermes Agent (inspired by Ralphy by michaelshimeles)
license: MIT
metadata:
  hermes:
    tags: [prd, task-loop, autonomous, opencode, coding-agent, parallel]
    related_skills: [opencode, claude-code, codex, writing-plans]
---

# PRD Task Loop

Autonomous development loop: read a PRD → execute tasks via AI agents → auto-commit → mark complete → repeat.

## When to Use

- User has a PRD/task list and wants autonomous execution
- User says "run all tasks" or "implement the PRD"
- User wants parallel worktree-based task execution
- User wants to compare AI coding agents on the same task list

## Task Sources

### Markdown PRD (default)
```markdown
## Features
- [ ] Create user model
- [ ] Add auth endpoints
- [x] Setup project (done)
```

### YAML Tasks
```yaml
tasks:
  - title: Create user model
    completed: false
  - title: Add auth endpoints
    completed: false
    parallel_group: 1
```

## Core Loop (Sequential)

1. Parse PRD → get next incomplete task
2. Build prompt with task + project context + rules
3. Run AI agent (opencode/claude/codex) with the prompt
4. Verify: files changed, tests pass (if applicable)
5. Mark task complete in PRD (`- [ ]` → `- [x]`)
6. Git commit with descriptive message
7. Repeat until all tasks done

## Usage

### Quick Start (Single Task)

```python
# In execute_code, use the run_task function:
from hermes_tools import terminal, write_file, read_file

task = "Create a utils.js file with an add(a,b) function"
result = terminal(command=f"opencode run '{task}'", workdir="~/project")
```

### Full PRD Loop (Sequential)

```
1. Read the PRD file with read_file(path="PRD.md")
2. Parse incomplete tasks (lines matching "- [ ]")
3. For each task:
   a. Build prompt (see Prompt Construction below)
   b. Run: terminal(command="opencode run '<prompt>'", workdir=project_dir)
   c. Verify output, check for errors
   d. Patch PRD: replace "- [ ] " with "- [x] " for completed task
   e. Git commit: terminal(command="git add -A && git commit -m 'feat: <task>'")
4. Report summary
```

### Parallel Execution (Worktree Isolation)

For tasks with `parallel_group` in YAML, or when user requests `--parallel`:

```
1. Create worktrees for each parallel task:
   terminal(command="git worktree add /tmp/ralphy-agent-1 feature-branch-1")
2. Run each agent in its worktree simultaneously:
   terminal(command="opencode run '<task>'", workdir="/tmp/ralphy-agent-1", background=true)
   terminal(command="opencode run '<task>'", workdir="/tmp/ralphy-agent-2", background=true)
3. Wait for completion: process(action="wait", session_id=...)
4. Merge branches back: terminal(command="git merge feature-branch-1")
5. Clean up worktrees: terminal(command="git worktree remove /tmp/ralphy-agent-1")
```

## Prompt Construction

Build a structured prompt for each task:

```
## Project Context
{name: "...", language: "...", framework: "..."}

## Rules (you MUST follow these)
{rules from .ralphy/config.yaml or user preferences}

## Boundaries - Do NOT modify
{files/patterns to never touch}

## Current Task
{task description from PRD}

## Instructions
1. Implement the task above
2. Write tests if applicable
3. Run tests and ensure they pass
4. Commit your changes with a descriptive message
ONLY WORK ON THIS SINGLE TASK.
```

## PRD Parsing

### Markdown Parser

```python
import re

def get_next_task(prd_content):
    for line in prd_content.split('\n'):
        match = re.match(r'^- \[ \] (.+)', line)
        if match:
            return match.group(1)
    return None

def mark_complete(prd_content, task):
    return prd_content.replace(f'- [ ] {task}', f'- [x] {task}', 1)

def count_remaining(prd_content):
    return len(re.findall(r'^- \[ \]', prd_content, re.MULTILINE))
```

### YAML Parser

```python
import yaml

def get_tasks(yaml_content):
    data = yaml.safe_load(yaml_content)
    return [t for t in data.get('tasks', []) if not t.get('completed')]

def get_parallel_group(tasks, group_num):
    return [t for t in tasks if t.get('parallel_group') == group_num]
```

## Engine Selection

| Engine | Command | Best For |
|--------|---------|----------|
| opencode | `opencode run '<prompt>' --format json` | Default choice, provider-agnostic |
| claude | `claude --dangerously-skip-permissions -p '<prompt>'` | Anthropic models, complex reasoning |
| codex | `codex exec --full-auto --json '<prompt>'` | OpenAI models, fast execution |

### Model Override

```bash
opencode run '<prompt>' --model openrouter/anthropic/claude-sonnet-4
opencode run '<prompt>' --model opencode/glm-4.7-free  # free tier
```

## Progress Tracking

Maintain a progress file (`.ralphy/progress.txt`) that agents append to:

```markdown
## Task 1: Create user model
- Created src/models/user.js with email and password fields
- Added bcrypt for password hashing
- Committed: abc1234

## Task 2: Add auth endpoints
- Created POST /api/signup and POST /api/login
- Added JWT token generation
- Committed: def5678
```

## Completion Detection

After each agent run, check:
1. Did files change? `git diff --name-only`
2. Did the agent claim completion? Look for `<promise>COMPLETE</promise>` or similar
3. Are there remaining tasks? `count_remaining(prd_content)`

## Configuration

### `.ralphy/config.yaml` (optional)

```yaml
project:
  name: "my-app"
  language: "TypeScript"
  framework: "Next.js"

commands:
  test: "npm test"
  lint: "npm run lint"
  build: "npm run build"

rules:
  - "use strict TypeScript"
  - "follow existing code patterns"

boundaries:
  never_touch:
    - "src/legacy/**"
    - "*.lock"
```

## Error Handling

- **Agent fails**: Retry up to 3 times with delay
- **Tests fail**: Report error, skip to next task (don't mark complete)
- **Merge conflicts**: Use AI agent to resolve, or fall back to manual
- **No more tasks**: Exit loop, show summary

## Summary Report

After loop completes, show:
- Tasks completed / total
- Git commits made
- Time elapsed
- Token usage / cost (if available)
- Any failed tasks

## Pitfalls

- OpenCode `run` mode doesn't support `@file` syntax (embed content in prompt instead)
- Don't run parallel agents in the same directory (use worktrees)
- PRD must be in the git repo root or agents won't find it
- Free models may timeout on complex tasks; set appropriate timeouts
- Git must be initialized before running

## Comparison: Ralphy Bash vs Hermes Skill

| Aspect | Ralphy (bash) | Hermes Skill |
|--------|--------------|--------------|
| Setup | `npm i -g ralphy-cli` | No install, uses existing tools |
| Flexibility | Fixed bash logic | Python/JS scripts, conditional logic |
| Parallel | Git worktrees built-in | `delegate_task` + worktrees |
| Integration | Standalone CLI | Part of agent ecosystem |
| Debugging | Shell scripts | Full tool access |
| Multi-engine | Flag-based switching | Skill-based switching |
