# Plan: Backport Argo Framework Learnings to Template Repo

## Overview

Port generic, battle-tested improvements from the Argo project's Claude configuration back to the public template repository (claude-agentic-framework). All Argo-specific details must be scrubbed.

## Scope

Medium Feature — 18 tasks across 5 work streams (Task 12 merged into Task 14).

## Accurate Diff Analysis

### Files That Are IDENTICAL (no work needed)

- **AGENTS.md** — Both repos have the same "Landing the Plane" protocol (40 lines, byte-for-byte identical)

### Template AHEAD of Argo (keep as-is)

- **Commands**: Template has 9 personas (architect, builder, qa-engineer, security-auditor, ui-ux-designer, code-check, swarm-plan, swarm-execute, swarm-review). Argo only has 6 (dropped architect, builder, qa-engineer, security-auditor into skills).
- **Hooks**: Template has `post-tool-use-beads-tracker.sh` and `subagent-stop-validator.sh` that Argo lacks
- **settings.json**: Template has more granular Bash permission patterns (e.g., `Bash(git status:*)` vs `Bash(git *)`) and SubagentStop hook
- **security.md**: Template has OWASP Top 10 table, Severity Classification, CWE References, Dependency Safety, Output Guidelines — Argo's security.md is more compact
- **code-quality.md**: Template has Performance Checklist, Two Hats Rule, Type Safety, DRY (knowledge vs incidental) — these are absent in Argo
- **.mcp.json**: Template has GitHub MCP server (Argo doesn't)

### Argo AHEAD of Template (port back)

| File | Argo Improvement | Risk |
|------|-----------------|------|
| CLAUDE.md | "Research Before Assuming", Working Directories, MCP Tools table, leaner structure | Medium — must generalize library names |
| core-directives.md | Decision Hierarchy, 7 numbered rules, Purpose/Constraints/Output sections | Low — entirely generic |
| code-quality.md | Naming conventions, structured logging, automated quality suite | Medium — must strip Argo-specific tools |
| security.md | "No Silent External Data Routing" rule, automated check annotations | Low — generic security principle |
| swarm-research.md | New research orchestrator command | Medium — must strip Argo output paths |
| Agents (all 5+1) | `permissionMode:` replaces `tools:`, skills refs, tool use rules, worker-research (new), reviewer→opus, remove focus modes (DRY with swarm-workers.md), better descriptions | Medium — must strip argo-* skills |
| Commands (all 9) | Remove `allowed-tools:`, add output destination, update handoffs, swarm-plan Product Planning | Low — generic simplification |
| Templates dir | 14 meta-templates (agent, command, skill, rule, hook + 9 artifacts) | Low — inherently generic |
| dangerous-command-guard.sh | Safety hook for --force-with-lease, hard reset, rm -rf, terraform | Low — generic |
| settings.json | $schema, alwaysThinkingEnabled, cleanupPeriodDays, SessionStart compact/clear matcher, sensitive path denials, WebFetch domains | Low — generic |
| .mcp.json | filesystem MCP, terraform MCP | Low — generic |
| skill-rules.json | Stripped `intent_patterns` and `file_patterns` (keywords only), added version notes | Low — pure simplification |
| SKILL.md files (all) | Removed `allowed-tools:` from frontmatter, slimmed verbose skills, added scratchpad pattern | Low — consistent with agent/command simplification |

## Redaction Rules

DO NOT port:

- Library names: Drizzle, Hono, Auth0, Stripe, Strands SDK, BAML, Textract
- Skill refs: argo-testing, argo-llm-integration, argo-ingestion, argo-agent-patterns, argo-shared-patterns
- Helpers: resolveAuthUser, errorResponse, useAccessToken, PaginatedResult
- Infra: Honeycomb, Aurora PostgreSQL, Bedrock guardrails, CloudWatch
- Output paths: `packages/api/research/output/`, `packages/api/src/utils/logger.ts`
- MCP servers: argo-debug, honeycomb, devservers
- Domain terms: life-file, claim-bundles, homeowners, carriers

## Tasks

### Stream A: Core Framework Files

#### Task 1: Refine CLAUDE.md

Add from Argo (generalized):

- "Research Before Assuming" section — generalize to "fast-evolving libraries" without naming specific ones
- "Working Directories" section — scratchpad/ (ephemeral) vs artifacts/ (durable)
- "MCP Tools" table — map tasks to generic tools
- Consider slimming by moving Artifacts table to core-directives.md (Argo's approach)

Keep from template:

- Core Principles (7) — Argo lacks these, they're valuable
- Personas table — Argo lacks this, useful for discoverability
- Beads section

Acceptance: Has new sections, no Argo specifics, still readable

#### Task 2: Enhance core-directives.md

Current template: 10 lines (just Skill-First + Protocol Adherence)
Target: ~70 lines matching Argo's structure

Add:

- Purpose section
- Constraints section (branch from main, verify artifacts, consult tech-strategy)
- 7 numbered rules (tech strategy authority, check skills, write tests, durable artifacts in ./artifacts/, ephemeral notes in ./scratchpad/, follow planning flow, follow persona protocols)
- Decision Hierarchy (Security > Tech Strategy > Core Directives > Skill Conventions > Local Judgment)
- Output section (commits, artifacts, scratchpad, handoffs)

Acceptance: Decision hierarchy documented, all 7 rules present, no Argo specifics

#### Task 3: Enhance code-quality.md

Add from Argo (generalized):

- Naming conventions section — generic defaults (camelCase files, PascalCase components, kebab-case multi-word dirs), marked as "suggested defaults — customize per project"
- Structured logging section — "Use a structured logger instead of raw console.* calls" (no specific module paths)
- Automated quality suite section — generic categories (pre-commit: lint+format+types+secrets, CI: lint+secrets+vuln scan, continuous: dependency updates) without naming specific tools

Keep from template (Argo lacks these):

- DRY (knowledge vs incidental distinction)
- Performance Checklist
- Quality Gates
- Two Hats Rule
- Type Safety

Acceptance: New sections present, no tool-specific names, template's unique sections preserved

#### Task 4: Enhance security.md

Add from Argo:

- "No Silent External Data Routing" rule (Rule 5 in Argo) — generic, important
- Automated check annotations on checklist items (e.g., "secretlint pre-commit + CI" → generalized to "pre-commit secret scanner + CI")

Keep from template (more comprehensive):

- OWASP Top 10 table
- Severity Classification table
- CWE References section
- Dependency Safety section
- Output Guidelines

Acceptance: Data routing rule present, checklist references automation, all existing sections preserved

### Stream B: Commands & Agents

#### Task 5: Add swarm-research.md command

Port Argo's research orchestrator, generalized:

- Remove `packages/api/research/output/` paths → use `scratchpad/` and `artifacts/`
- Remove "Argo reference knowledge pipeline" from default template
- Remove "homeowners" from executive summary guidance
- Remove carrier/brand-specific cross-checking examples → generalize to "domain-specific entities"
- Keep: workflow, dispatch modes, scope splitting, verification tiers, confidence levels, cross-checking protocol, quality gates, synthesis format, Beads integration

Note: Task 5 runs in parallel with Task 7b (which updates handoffs in existing commands).
Include correct handoff targets (swarm commands, not persona names) at creation time.

Acceptance: Command works, no Argo paths/terms, research methodology intact, handoffs
reference swarm commands

#### Task 6: Add worker-research.md agent

Port Argo's deep research agent, generalized. This is a NEW agent alongside the existing
worker-researcher (sonnet, quick lookups). worker-research is opus-level deep investigation.

Strip:

- "Argo research prompts" from confidence calibration
- Pipeline metadata references ("target domains/tags")
- "Argo reference knowledge pipeline" from default template header
- "homeowners" from executive summary guidance
- `packages/api/research/` paths

Keep:

- 5-phase methodology (Decompose → Survey → Analyze → Deep Dive → Synthesize)
- Verification tiers (web-required, web-preferred, training-acceptable)
- Confidence calibration (CONFIDENT / HEDGED / DEFER + HIGH / MEDIUM / LOW)
- Source verification rules, anti-patterns, tool use rules, self-check

Acceptance: Agent file exists with opus model, methodology intact, no Argo terms

#### Task 7: Upgrade existing agent definitions

This is a structural simplification learned from Argo. Three changes across all agents:

**Change 1: Replace `tools:` with `permissionMode: acceptEdits`**

Argo dropped explicit `tools:` allowlists from all agents. Instead of restricting which tools
an agent can use, it sets `permissionMode: acceptEdits` and relies on agent instructions to
guide behavior. This makes agents more capable (e.g., explorer can do web research) without
changing frontmatter when new tools are needed.

Before (template):
```yaml
tools: Read, Glob, Grep
model: haiku
```

After (Argo pattern):
```yaml
model: haiku
permissionMode: acceptEdits
```

**Change 2: Add `skills:` references (generic only)**

Argo adds `skills:` to inject domain knowledge at startup. Agents don't inherit skills
from parent — they must be listed explicitly.

| Agent | Skills to add |
|-------|--------------|
| worker-reviewer | `application-security` |
| worker-architect | `designing-systems, writing-adrs, designing-apis` |
| worker-builder | (none — template has no argo-specific skills to add) |
| worker-explorer | (none) |
| worker-researcher | (none) |

**Change 3: Add "Tool Use Rules" section to all agents**

Prevents a real permission-matching bug where agents write `# comment\ncommand` in Bash
calls, which doesn't match `Bash(git status:*)` patterns.

```markdown
## Tool Use Rules

- **Never prefix Bash commands with shell comments** (`# comment\ncommand`).
  This breaks permission auto-approval pattern matching.
- Prefer dedicated tools (Read, Grep, Glob) over Bash equivalents.
```

**Per-agent specific changes:**

- **worker-explorer.md**: Improve description to "Codebase exploration and web research
  worker. Use for pattern search, dependency mapping, doc lookup, and library comparison."
  Add web research to capabilities. Add Sources to output format.

- **worker-builder.md**: Improve description to "Implementation, testing, and refactoring
  worker for swarm tasks. Use for parallel coding, test writing, and code cleanup."
  Remove "Focus Modes" section (duplicates swarm-workers.md rule; orchestrator passes focus
  via prompt). Keep constraints.

- **worker-reviewer.md**: Upgrade model from sonnet to opus (security review benefits from
  stronger reasoning). Improve description to "Code review, security audit, and QA worker
  for swarm tasks. Use for parallel review, vulnerability detection, and quality assessment."
  Remove "Focus Modes" section. Add Security Severity Levels (CRITICAL/HIGH/MEDIUM/LOW/INFO).
  Add CWE reference in output format.

- **worker-architect.md**: Improve description (already good). Add skills refs.

- **worker-researcher.md**: Keep as-is (sonnet, quick web lookups). Improve description to
  "Quick web research and documentation lookup. Use for API docs, library comparison, and
  best practices." This complements the new worker-research (opus, deep investigation).

Acceptance: All agents use permissionMode (no tools:), have tool use rules, reviewer is opus,
descriptions include "Use for..." routing triggers, focus modes removed from builder/reviewer.

#### Task 7c: Update swarm-workers.md to match agent changes

swarm-workers.md is the single source of truth for worker definitions. Tasks 6 and 7 change
what it describes, so it must be updated to stay consistent.

Changes:

- **Worker Types table**: Replace `Tools` column with `Primary Use` (tools are no longer
  restricted via frontmatter — `permissionMode: acceptEdits` gives full access). Update
  reviewer model from `sonnet` to `opus`. Add `worker-research` row (opus, deep multi-source
  investigation).
- **Context Efficiency**: Reword point 3 from "Minimal tools — Only tools needed for the task"
  to "Guided behavior — Agent instructions define scope, permissionMode controls access"
- **Context Efficiency**: Reword point 1 from "workers use focused tool sets" to "workers use
  focused instructions"
- **Anti-Patterns**: Keep "NO opus for simple tasks" — still valid for cost optimization

Keep unchanged:

- Worker Focus Modes section (single source of truth — agents no longer duplicate this)
- Coordination Protocol
- Swarm Patterns
- Performance Tips

Acceptance: Worker Types table matches actual agent definitions, no stale model or tools
references, worker-research is listed.

#### Task 7b: Simplify and enhance commands

Same `allowed-tools:` simplification as agents, plus targeted content additions.

**Change 1: Remove `allowed-tools:` from all commands**

Five commands restrict tools in frontmatter. Argo removed all of these. Same rationale as
agents — persona instructions guide behavior, tool restrictions are brittle and limiting.

Commands to update:

- `architect.md` — remove `allowed-tools: Read, Glob, Grep, Write, Task, mcp__sequential-thinking__*`
- `builder.md` — remove `allowed-tools: Read, Write, Edit, Bash, Glob, Grep, Task, mcp__github__*`
- `qa-engineer.md` — remove `allowed-tools: Read, Write, Edit, Bash, Glob, Grep, mcp__chrome-devtools__*`
- `security-auditor.md` — remove `allowed-tools: Read, Glob, Grep, Bash, mcp__sequential-thinking__*, mcp__github__*`
- `ui-ux-designer.md` — remove `allowed-tools: Read, Write, Glob, Grep, mcp__chrome-devtools__*`

**Change 2: Add scratchpad/ vs artifacts/ output guidance**

Add brief output destination note to commands that produce artifacts but don't mention
scratchpad/. Working notes go to `scratchpad/`, final docs to `artifacts/`.

Commands to update: architect, builder, qa-engineer, security-auditor, ui-ux-designer

**Change 3: Update handoff targets to use swarm commands**

Template handoffs reference individual personas ("To Builder", "To Architect"). Argo uses
swarm commands ("To swarm-execute", "To swarm-review"). Swarm commands are the actual
entry points for the next phase.

Commands to update: all 9 — replace persona names in handoffs with swarm commands.

**Change 4: swarm-plan — add Product Planning section**

Port from Argo (generalized). Add a section covering the planning flow with references to
existing skills rather than embedding template content:

- PR-FAQ structure → reference `writing-pr-faqs` skill
- PRD structure → reference `writing-prds` skill
- Architecture Design process (Understand → Reason → Design → Validate)
- scratchpad/ for planning exploration, artifacts/ for final documents
- More related skills: `designing-systems`, `designing-apis`, `writing-pr-faqs`, `writing-prds`, `requirements-analysis`

Do NOT inline ADR or PRD templates (those belong in skills).

**Change 5: swarm-review — add smart constraint**

Add: "Never recommend removing a symbol without verifying all references via Grep first"

**What NOT to change:**

- Template's DRY approach of referencing rules (code-quality.md, security.md, swarm-workers.md)
  is BETTER than Argo's approach of duplicating content inline. Keep it.
- Keep all 9 commands (Argo dropped architect, builder, qa-engineer, security-auditor — but
  the template's dual approach of individual personas + swarm orchestrators is more flexible)
- Do NOT port Argo's swarm-execute Build & Deploy Reference (100+ lines of project-specific
  pnpm/docker/terraform commands)
- Do NOT port inline STRIDE, performance checklist, or SOLID signals into commands — they
  belong in rules

Acceptance: No command has `allowed-tools:`, all have output destination guidance, handoffs
reference swarm commands, swarm-plan has Product Planning section.

### Stream C: Hooks Hardening

#### Task 8: Remove `set -e` / `set -eo pipefail` from all hooks

This is the highest-impact reliability fix. `set -e` in hooks is dangerous:

- A jq parse returning empty causes exit
- A `bd` command failing (daemon down) kills the whole hook
- `grep` returning no matches exits non-zero

Argo learned this the hard way and removed `set -e` from every hook, using `|| exit 0` or `|| true` on individual commands instead.

Files to fix (all have `set -e` or `set -eo pipefail`):

- session-start-loader.sh (`set -eo pipefail`)
- pre-tool-use-validator.sh (`set -eo pipefail`)
- pre-commit-verification.sh (`set -e`)
- pre-push-main-blocker.sh (`set -e`)
- post-tool-use-tracker.sh (`set -e`)
- post-tool-use-beads-tracker.sh (`set -e`)
- stop-validator.sh (`set -e`)
- subagent-stop-validator.sh (`set -e`)

Replace with per-command error handling: `|| exit 0`, `|| true`, `2>/dev/null`.

Acceptance: No hook uses `set -e`. Each command handles its own errors.

#### Task 9: Merge post-tool-use hooks into one (DRY)

Template runs TWO PostToolUse hooks per file edit:

- `post-tool-use-tracker.sh` (47 lines) — basic logging, skips .md, 100-entry limit
- `post-tool-use-beads-tracker.sh` (78 lines) — session logging, lock release, Beads, 500-entry limit

They duplicate: relative path computation, tracker writes, file logging. Argo merged these into one 61-line hook.

Action:

- Merge into single `post-tool-use-tracker.sh` with all features:
  - Session-aware logging (from beads-tracker)
  - Lock release (from beads-tracker)
  - Beads integration (from beads-tracker)
  - Skip patterns using case statement (faster than array loop)
  - Hysteresis truncation: only truncate at 600→500 (not every write)
- Remove `post-tool-use-beads-tracker.sh`
- Update settings.json to register only one PostToolUse hook

Acceptance: Single PostToolUse hook, all features preserved, settings.json updated.

#### Task 10: Harden session-start-loader.sh

Note: `set -e` removal is already handled by Task 8 (dependency). This task covers additional
hardening only.

Port from Argo:

- Add `timeout 3` to `bd ready` and `bd list` calls (prevents hang if Beads daemon is stuck)
- Add lock cleanup at session start (stale locks >5 min, matching Argo's pattern)
- Use `mkdir -p "$STATE_DIR" "$LOCK_DIR"` (ensure lock dir exists)

Keep from template:

- `find -mtime +1 -delete` for session cleanup (cleaner than manual loop)
- Additional swarm status hints ("Use bd create", "Check file locks")

Acceptance: bd calls have timeout, stale locks cleaned.

#### Task 11: Add dangerous-command-guard.sh hook

Port from Argo as-is (already generic). Register in settings.json under PreToolUse Bash.

### Stream D: Templates & Configuration

#### Task 13: Create templates directory

Port all 14 templates from Argo (already generic):

- `.claude/templates/agent.template.md` — as-is
- `.claude/templates/command.template.md` — as-is
- `.claude/templates/skill.template.md` — as-is
- `.claude/templates/rule.template.md` — as-is
- `.claude/templates/hook.template.sh` — as-is (comprehensive hook schema docs)
- `.claude/templates/artifacts/adr.template.md`
- `.claude/templates/artifacts/design_spec.template.md`
- `.claude/templates/artifacts/plan.template.md`
- `.claude/templates/artifacts/postmortem.template.md`
- `.claude/templates/artifacts/pr_faq.template.md`
- `.claude/templates/artifacts/prd.template.md`
- `.claude/templates/artifacts/roadmap.template.md`
- `.claude/templates/artifacts/security_audit.template.md`
- `.claude/templates/artifacts/system_design.template.md`

Acceptance: All 14 files exist, no Argo references

#### Task 14: Update settings.json

Add from Argo:

- `"$schema": "https://json.schemastore.org/claude-code-settings.json"`
- `"alwaysThinkingEnabled": true`
- `"cleanupPeriodDays": 90`
- SessionStart matcher: `"startup|resume|compact|clear"` (currently only `"startup|resume"`)
- PreToolUse Write matcher: add `NotebookEdit` → `"Write|Edit|MultiEdit|NotebookEdit"`
- Dangerous-command-guard hook registration under PreToolUse Bash
- Sensitive path read denials from Argo (~/.ssh, ~/.gnupg, ~/.aws, ~/.azure, etc.)
- Edit denials for shell profiles (~/.bashrc, ~/.zshrc, ~/.bash_profile, ~/.ssh/**)
- WebFetch domain permissions (move from allow list if needed)
- Pipe-to-bash denial: `Bash(curl * | bash*)`, `Bash(wget * | bash*)`

Keep from template:

- Granular Bash patterns (better than Argo's wildcard approach)
- SubagentStop hook (Argo lacks this)
- Comment organization with `// ===` sections

Acceptance: All new settings present, existing granular permissions preserved

#### Task 15: Update .mcp.json

Add:

- `filesystem` MCP server (official Anthropic: `@modelcontextprotocol/server-filesystem`)
- `awslabs.terraform-mcp-server` (generic IaC tool)

Keep:

- sequential-thinking, chrome-devtools, github, context7

DO NOT add: argo-debug, honeycomb, devservers

Acceptance: 6 MCP servers configured, no Argo-specific servers

### Stream E: Skills Simplification

#### Task 16: Simplify skill-rules.json

Argo v2.5 stripped skill-rules.json down to keywords-only triggers. The `intent_patterns` and
`file_patterns` fields exist in every template skill entry but are dead weight — the
skill-activation hook only processes keywords.

Changes:

- Remove `intent_patterns` from all 57 skill entries
- Remove `file_patterns` from all 57 skill entries (keep `keywords` only)
- Add `notes` field to metadata for version tracking (e.g., "v2.1: Stripped dead trigger fields")
- Update `version` from "2.0.0" to "2.1.0"

This cuts skill-rules.json from ~725 lines to ~400 lines — 45% smaller, faster to parse.

Acceptance: No skill has intent_patterns or file_patterns, notes field present, version bumped.

#### Task 17: Simplify SKILL.md files

Same simplification pattern as agents/commands — drop `allowed-tools:` from frontmatter.
Argo removed this from all 38 skills. Template has it in 51 of 77 skills.

Changes:

- Remove `allowed-tools:` line from all 51 SKILL.md frontmatter blocks
- Remove `## Overview` boilerplate sections that add no value (19 skills have these —
  replace with a direct opening sentence or remove entirely when the content starts immediately)
- Remove `## MCP Tools` sections from beads-workflow and testing skills (verbose GitHub MCP
  references that don't add value — the MCP tools are available regardless)
- Add `scratchpad/` pattern to swarm-coordination skill: "Workers write results to
  `scratchpad/<task-id>.md`, not direct context. Only durable artifacts go to `artifacts/`."
- Slim swarm-coordination and beads-workflow by removing boilerplate (Overview sections,
  verbose code block examples, redundant explanations). Target ~100 lines for swarm-coordination
  and ~120 lines for beads-workflow. Do NOT cut reference tables (Issue Types, Priority Levels),
  Troubleshooting sections, or Emergency Procedures — template users need this onboarding content
  that Argo's experienced team could drop.

Acceptance: No skill has `allowed-tools:`, no gratuitous Overview sections, scratchpad
pattern documented in swarm-coordination, reference tables and troubleshooting preserved.

## Dependency Graph

```text
Phase 1a (parallel, 8 workers):
  Task 2  (core-directives.md)
  Task 3  (code-quality.md)
  Task 4  (security.md)
  Task 5  (swarm-research.md command)
  Task 6  (worker-research.md agent)
  Task 7  (agent upgrades — all 5 agents)
  Task 8  (remove set -e from all hooks)
  Task 13 (templates directory)

Phase 1b (parallel, 5 workers — starts as Phase 1a workers free up):
  Task 7b (command simplification)
  Task 7c (swarm-workers.md — depends on Task 7 for final agent state)
  Task 15 (.mcp.json)
  Task 16 (skill-rules.json simplification)
  Task 17 (SKILL.md simplification)

Phase 2 (parallel, 3 workers — depends on Task 8):
  Task 9  (merge post-tool-use hooks) — reads fresh file state after Task 8
  Task 10 (harden session-start) — set -e already removed by Task 8
  Task 11 (dangerous-command-guard)

Phase 3 (sequential — depends on Phase 2):
  Task 14 (settings.json) — consolidates all settings: hook registrations, schema,
           alwaysThinking, SessionStart matcher, sensitive path denials, merged hook ref
  Task 1  (CLAUDE.md) — references rules updated in Tasks 2-4
```

## Execution Strategy

Phase 1a (parallel, 8 workers): Tasks 2-8, 13
Phase 1b (parallel, 5 workers): Tasks 7b, 7c, 15, 16, 17
Phase 2 (parallel, 3 workers): Tasks 9-11
Phase 3 (sequential): Tasks 14, 1

Use `/swarm-execute` with workers for each phase.

## Design Decisions

### Agent focus modes: removed from agents, kept in swarm-workers.md

Template duplicates focus mode docs in two places: agent files (worker-builder, worker-reviewer)
AND .claude/rules/swarm-workers.md. Argo removed them from agent files — the orchestrator
(swarm-execute, swarm-review) already knows which mode to use and passes it in the prompt.

Decision: **Remove focus modes from agent files.** swarm-workers.md remains the single source
of truth for worker types, focus modes, and coordination patterns. This is DRY.

### Agent tool restrictions: `permissionMode:` over `tools:`

Template uses `tools:` allowlists to restrict each agent's capabilities. Argo dropped this
in favor of `permissionMode: acceptEdits` and relies on agent instructions for behavior.

Decision: **Use Argo's approach.** Tool restrictions are brittle (break when new tools are
added) and limit agent capability unnecessarily (explorer can't web search). Permission mode
is the right security boundary; agent instructions are the right behavior boundary.

**Customization point**: Template users who want stricter agent control can restore `tools:`
in agent frontmatter. The `permissionMode: acceptEdits` default is the recommended starting
point — it auto-approves file edits but still requires approval for other actions.

### worker-researcher vs worker-research: keep both

Argo replaced worker-researcher with worker-research entirely. But these serve different needs:
- worker-researcher (sonnet): quick API doc lookups, library comparisons, 33 lines
- worker-research (opus): deep multi-source investigation with 5-phase methodology, 200+ lines

Decision: **Keep both.** Template users get a lightweight option for quick lookups and a
heavyweight option for deep research. Orchestrators choose based on task complexity.

### Commands: DRY references over inline duplication

Argo's swarm-review embeds a full STRIDE walkthrough, inline performance checklist, severity
table, and infrastructure review — all content that already lives in security.md and
code-quality.md. Similarly, Argo's code-check embeds detailed SOLID signals, observability
audit, and TS/React smells. This creates maintenance burden when rules change.

Template's approach of referencing rules ("See security.md for OWASP Top 10", "Apply SOLID
principles from code-quality.md") is better. Commands should reference rules for shared
standards and only contain content unique to the persona's workflow.

Decision: **Keep template's DRY references.** Do NOT port Argo's inline duplications.

### Commands: keep all 9 personas

Argo dropped architect, builder, qa-engineer, security-auditor (4 commands). The template's
approach of having both individual personas AND swarm orchestrators is more flexible — users
can either `/architect` for a focused session or `/swarm-plan` → worker-architect for
parallel design.

Decision: **Keep all 9 commands.** The dual approach is a template strength.

### Skills: keywords-only triggers, drop allowed-tools

Template skill-rules.json has `intent_patterns` and `file_patterns` in every trigger entry,
but the skill-activation hook only processes `keywords`. Argo v2.5 removed these dead fields
entirely. Similarly, `allowed-tools` in SKILL.md frontmatter was dropped — same pattern as
agents and commands.

Decision: **Strip dead fields.** Keywords-only triggers are simpler and sufficient. Skills
don't need tool restrictions any more than agents do.

### Skills: keep template's broader skill set (77 vs Argo's 38)

Argo pruned to 38 skills (10 of which are Argo-specific). Template has 77 generic skills
covering more languages, frameworks, and workflows. As a public template, broader coverage
is a feature — users remove what they don't need.

Decision: **Keep all 77 skills.** Only simplify structure, don't reduce count.

## Design Decisions (Not Porting)

### skill-activation-prompt: TypeScript vs Bash

Argo rewrote the skill activation hook in pure bash (88 lines, single jq pass). It also **loads actual skill file content** into `<system-reminder>` tags. The template uses TypeScript (162 lines) with richer matching (regex intent_patterns) but only outputs skill name suggestions — it doesn't inject content.

Decision: **Keep TypeScript version** for now. The regex intent matching is a better UX for a template framework where users will customize skills. However, consider a future task to make it inject skill content (not just suggest) — that's Argo's real win.

### post-tool-use-tracker: .md file skip

Template's basic tracker skips `.md` files. Argo's merged tracker doesn't skip by extension — it skips by directory pattern (.claude/hooks/, .beads/, .git/, node_modules/) and by extension (.log, .lock). The directory-based approach is better. The .md skip is too broad (skips tracking edits to ADRs, plans, etc.).

Decision: **Use Argo's skip patterns** in the merged hook. Remove the .md skip.

## Handoff

To `/swarm-execute`: This plan + Beads issues ready via `bd ready`
