# Plan: Token Usage Optimization

**Status**: Ready for implementation
**Scope**: Small feature (1-3 days)
**Decision type**: Two-Way Door (all changes reversible)

## Corrected Token Budget

Research revealed the initial analysis overcounted. Here's the corrected picture:

### What's Actually in LLM Context (Every Turn)

| Component | ~Tokens | Notes |
|-----------|--------:|-------|
| CLAUDE.md | 1,187 | Loaded every turn |
| rules/core-directives.md | 995 | Auto-loaded |
| rules/swarm-workers.md | 925 | Auto-loaded (wasted on non-swarm) |
| rules/tech-strategy.md | 839 | Auto-loaded **AND** @-imported = **double-loaded** |
| rules/security.md | 736 | Auto-loaded |
| rules/code-quality.md | 689 | Auto-loaded |
| Native skill descriptions | ~1,675 | ~67 skills × ~25 tokens each |
| Hook output (skill suggestions) | ~100-400 | Per-prompt, varies by match count |
| **Total base context** | **~7,046–7,346** | Before any command/skill invocation |

### What's NOT in LLM Context (Correction)

| Component | Previous Estimate | Actual | Why |
|-----------|------------------:|-------:|-----|
| settings.json | 2,593 tokens | 0 | CLI-only, never sent to LLM |
| skill-rules.json | 5,479 tokens | 0 | Read from disk by hook, not injected |

**Previous analysis overestimated base context by ~8,072 tokens.** The real base is ~7,100 tokens, not ~13,400.

## Issues Found

### Issue 1: tech-strategy.md Double-Loading (CONFIRMED)

CLAUDE.md line 46 contains `@.claude/rules/tech-strategy.md`. Since all `rules/*.md` files auto-load, this file is loaded **twice** — costing ~839 wasted tokens every turn.

**Fix**: Remove the `@` import from CLAUDE.md. The auto-load handles it.
**Savings**: ~839 tokens/turn
**Risk**: None — content still loads via rules/

### Issue 2: swarm-workers.md Always Loaded

925 tokens loaded every session. Research confirmed:
- 70% of content is swarm-specific (worker types, focus modes, coordination protocol)
- 30% is universal (anti-patterns)
- Worker agents DON'T reference this file — they have self-contained instructions
- Only swarm-execute.md references it (2 links)

**Fix**: Split into two files:
- `rules/agent-constraints.md` — Anti-patterns only (~6 lines, ~50 tokens)
- Move worker types, focus modes, coordination protocol, performance tips into the swarm command files that actually use them

**Savings**: ~875 tokens/turn for non-swarm sessions
**Risk**: Low — swarm commands already contain most coordination context

### Issue 3: Oversized Language Skills (On-Demand, But Costly When Triggered)

Top 6 language skills average ~4,000 tokens each. Content analysis:
- 50-65% is copy-paste code examples
- 25% is actionable rules/patterns
- 10-15% duplicates what Context7 MCP can provide on-demand

| Skill | Current | Target | Approach |
|-------|--------:|-------:|----------|
| radix-ui | 5,409 | ~3,000 | Keep 1 example per component, trim styling |
| vite | 4,188 | ~2,500 | Trim config examples, reference Context7 |
| hono | 4,124 | ~2,500 | Trim API examples, keep patterns |
| react-patterns | 3,604 | ~2,200 | Consolidate hook examples |
| tailwind-css | 3,428 | ~2,000 | Trim utility reference, keep patterns |
| biome | 3,380 | ~2,000 | Trim CLI reference, keep config patterns |

**Savings**: ~8,000 tokens when skills are triggered (not per-turn)
**Risk**: Medium — must preserve enough context for offline-first usage. Do NOT remove patterns, only trim verbose code blocks and reference material that Context7 provides better.

### Issue 4: swarm-research Command + worker-research Agent Bloat

| File | Tokens | Issue |
|------|-------:|-------|
| commands/swarm-research.md | 3,100 | Largest command file, 3x average |
| agents/worker-research.md | 2,842 | 3x larger than other agent files |

worker-research.md is justified — it teaches verification methodology. But swarm-research.md likely contains content that could be in a skill.

**Fix**: Audit swarm-research.md for content that can move to the swarm-coordination skill.
**Savings**: ~1,000 tokens when /swarm-research invoked
**Risk**: Low

### Issue 5: CLAUDE.md Line Count

Currently 115 lines. Best practice is <200 lines. Not critical yet, but the Core Principles section (lines 12-42) repeats what `core-directives.md` and other rules already say.

**Fix**: Trim CLAUDE.md Core Principles to one-liners with "see rules/ for details". Keep the tables and Quick Reference.
**Savings**: ~400 tokens
**Risk**: Low — rules files carry the detail

### Issue 6: Hook Output Adds Per-Turn Cost

The skill-activation hook injects 3-4 lines per matched skill into every prompt response. With broad keywords like "api", "auth", "design", this can match 5+ skills = ~300 tokens of suggestions injected per turn.

**Fix**: Cap suggestions to top 3 by priority. Add early-exit for common false positives.
**Savings**: ~100-200 tokens on prompts that trigger many matches
**Risk**: None — fewer suggestions = less noise

## Implementation Tasks

### Task 1: Fix tech-strategy.md Double-Load
- Remove `@.claude/rules/tech-strategy.md` from CLAUDE.md line 46
- Replace with inline text: `See `.claude/rules/tech-strategy.md` for technology choices.`
- **Verify**: tech-strategy still loads via rules/ auto-load

### Task 2: Split swarm-workers.md
- Create `rules/agent-constraints.md` with universal anti-patterns only
- Move worker types table → swarm-execute.md, swarm-plan.md
- Move focus modes → swarm-execute.md
- Move coordination protocol → swarm-execute.md
- Move performance tips → swarm-execute.md, swarm-plan.md
- Delete `rules/swarm-workers.md`
- **Verify**: `/swarm-execute` still has all context it needs, grep for cross-references

### Task 3: Trim Top 6 Language Skills
- For each: keep workflow section, 1 complete example per major concept, anti-patterns
- Move verbose API reference/config blocks to `resources/` subdirectory or add Context7 hints
- Target: no skill >3,500 tokens (~14,000 bytes)
- **Verify**: each skill still has enough context to guide implementation without external lookup

### Task 4: Trim CLAUDE.md Core Principles
- Reduce 7 principles from paragraph to one-liner each
- Add "Details in `.claude/rules/`" footer
- Target: CLAUDE.md under 90 lines
- **Verify**: no loss of essential guidance

### Task 5: Cap Skill Suggestion Output
- In `skill-activation-prompt.ts`: limit output to top 3 matches by priority
- Skip suggestions when prompt is a slash command (already invoking a persona)
- **Verify**: critical/high matches still surface

### Task 6: Audit swarm-research.md
- Identify content duplicated with swarm-coordination skill
- Move reference material to skill, keep command file focused on orchestration flow
- **Verify**: /swarm-research workflow unchanged

## Expected Impact

| Scenario | Before | After | Savings |
|----------|-------:|------:|--------:|
| Solo session (base) | 7,100 | 5,600 | **1,500 tokens/turn** (21%) |
| Solo + 1 language skill | 10,100 | 8,100 | **2,000 tokens** |
| Swarm (4 workers) | 28,400+ | 22,400+ | **6,000+ tokens** |
| Swarm (8 workers) | 56,800+ | 44,800+ | **12,000+ tokens** |

## Dependency Order

```
Task 1 (double-load fix) — no dependencies, immediate
Task 4 (CLAUDE.md trim) — no dependencies, immediate
Task 5 (cap suggestions) — no dependencies, immediate
Task 2 (split swarm-workers) — requires cross-reference audit
Task 6 (swarm-research trim) — requires Task 2 complete
Task 3 (language skills) — independent, largest effort
```

## Verification Checklist

- [ ] All personas still invocable and functional
- [ ] All skills still triggerable via hooks
- [ ] Swarm workers still receive correct context
- [ ] No content deleted — only relocated or condensed
- [ ] `grep -r "swarm-workers" .claude/` returns no broken references
- [ ] tech-strategy.md appears exactly once in context (not twice)
