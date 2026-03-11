# Swarm Worker Guidelines

Rules for efficient multi-agent swarm execution.

## Context Efficiency

1. **Workers inherit session context** - CLAUDE.md and rules are loaded, but workers use focused instructions
2. **Narrow scope** - Each worker focuses on one task
3. **Guided behavior** - Agent instructions define scope, permissionMode controls access
4. **Right-sized models** - Haiku for exploration, Sonnet for implementation, Opus for architecture

## Worker Types

| Worker | Model | Primary Use |
|--------|-------|-------------|
| `worker-explorer` | haiku | Fast codebase search, web research, dependency mapping |
| `worker-builder` | sonnet | Implementation, testing, refactoring |
| `worker-reviewer` | opus | Code review, security audit, quality assessment |
| `worker-researcher` | sonnet | Quick web research, API docs, library comparison |
| `worker-research` | opus | Deep multi-source investigation, technology evaluation |
| `worker-architect` | opus | Complex design decisions, ADRs, system architecture |

## Worker Focus Modes

Orchestrators specialize workers by specifying a focus mode in the prompt.

**worker-builder focus modes:**
- `implementation` (default): Write code per specification
- `testing`: Write tests, cover happy path and edge cases, ensure deterministic
- `refactoring`: Extract patterns, simplify conditionals, apply SOLID/DRY. Follow Two Hats Rule (see code-quality.md)

**worker-reviewer focus modes:**
- `quality` (default): Code review checklist — naming, style, tests, patterns
- `security`: OWASP Top 10 scan, hardcoded secrets, auth/authz flows, input validation. Reference CWE IDs. See security.md
- `performance`: N+1 queries, blocking I/O, allocations, pagination, caching. See code-quality.md

## Swarm Patterns

### Parallel Exploration
```
Orchestrator spawns 4-8 worker-explorer agents simultaneously
Each searches different parts of codebase
Results aggregated for next phase
```

### Divide and Conquer
```
1. worker-architect designs solution
2. Orchestrator decomposes into N tasks
3. N worker-builder agents execute in parallel
4. worker-reviewer validates each output
5. Orchestrator integrates
```

### Security Sweep
```
worker-reviewer (focus: security) scans all components in parallel
Findings aggregated and prioritized
worker-builder fixes critical/high issues
```

## Coordination Protocol

1. **Orchestrator** decomposes task via Beads
2. **Workers** claim issues: `bd update <id> --status in_progress`
3. **Workers** complete task following AGENTS.md "Landing the Plane" workflow
4. **Workers** report completion to orchestrator
5. **Orchestrator** integrates and verifies

### Worker Completion Requirements

When a worker completes its assigned task, it MUST follow the full completion protocol from AGENTS.md:

1. File issues for remaining work
2. Run quality gates (if code changed)
3. Update issue status: `bd close <id>`
4. **PUSH TO REMOTE** (mandatory):
   ```bash
   git pull --rebase
   bd sync
   git push
   ```
5. Report completion to orchestrator

**Critical**: Workers must push changes to remote. Work is NOT complete until `git push` succeeds.

## Performance Tips

- Launch multiple explorers for broad searches
- Use worker-architect for decisions, worker-builder for execution
- Parallelize independent tasks (max 8 concurrent workers)
- Keep worker prompts under 500 tokens for fast startup

## Anti-Patterns

- NO loading full context into workers
- NO sharing state between workers (use Beads)
- NO workers spawning workers (single-level only)
- NO long-running workers (timeout at 5 min)
- NO opus for simple tasks (cost optimization)
- NO skipping git push (see Worker Completion Requirements above)
