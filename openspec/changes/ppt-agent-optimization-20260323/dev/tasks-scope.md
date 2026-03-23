# Dev Scope — Wave 1: Unlock Foundation

> Proposal: ppt-agent-optimization-20260323
> Scope: T-01 ~ T-06 (6 tasks, all effort S, zero behavioral risk)

## Selected Tasks

### T-01: Add Bash to review-core tools
- **File**: `agents/review-core.md`
- **Change**: Add `Bash` to `tools` list in frontmatter
- **Why**: Pre-review automated checks (xmllint, grep, etc.) are dead code without Bash
- **Acceptance**: `tools` array includes `Bash`

### T-02: content-core maxTurns 25→35
- **File**: `agents/content-core.md`
- **Change**: `maxTurns: 25` → `maxTurns: 35`
- **Why**: 12+ slide decks truncate when content-core runs out of turns
- **Acceptance**: `maxTurns: 35` in frontmatter

### T-03: Batch draft signals every 3 slides
- **Files**: `agents/content-core.md`, `commands/ppt.md`
- **Change**: content-core sends `draft_slide_ready` every 3 slides instead of per-slide; ppt.md updates pipeline description accordingly
- **Why**: Per-slide signaling consumes turns too fast with the turn budget
- **Acceptance**: content-core draft mode batches signals; ppt.md Phase 5/6 reflects batching

### T-04: slide-core maxTurns 30→20
- **File**: `agents/slide-core.md`
- **Change**: `maxTurns: 30` → `maxTurns: 20`
- **Why**: slide-core handles one slide per invocation; 30 turns is excessive
- **Acceptance**: `maxTurns: 20` in frontmatter

### T-05: review-core maxTurns 15→20
- **File**: `agents/review-core.md`
- **Change**: `maxTurns: 15` → `maxTurns: 20`
- **Why**: 15 turns is too tight for pre-review checks + Gemini call + structured output
- **Acceptance**: `maxTurns: 20` in frontmatter

### T-06: outline.json approved field + resume guard
- **Files**: `commands/ppt.md`, `skills/_shared/references/prompts/outline-architect.md`
- **Change**: Add `"approved": false` to outline.json schema; ppt.md resume logic checks `approved` field before skipping Phase 4
- **Why**: `--run-id` resume currently skips Phase 4 Hard Stop if outline.json exists, even if user never approved
- **Acceptance**: outline.json schema includes `approved` field; resume detection checks it

## Atomic Groups
- T-02 + T-03 (turn budget optimization is coupled)

## Test Expectations
- All changes are prompt/config-level — no runtime code to test
- Verify: frontmatter YAML is valid, JSON schema is consistent, instructions are unambiguous
