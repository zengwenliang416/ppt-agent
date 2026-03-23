# Dev Progress — PPT Agent Optimization

> Proposal: ppt-agent-optimization-20260323
> Date: 2026-03-23

## Wave 1: Unlock Foundation — COMPLETE

| Task | Change | File(s) | Verified |
|------|--------|---------|----------|
| T-01 | Added `Bash` to review-core tools | `agents/review-core.md` | frontmatter valid |
| T-02 | maxTurns 25→35 | `agents/content-core.md` | frontmatter valid |
| T-03 | Per-slide → batch-of-3 signaling (`draft_slides_ready`) | `agents/content-core.md`, `commands/ppt.md` | signal name consistent across both files |
| T-04 | maxTurns 30→20 | `agents/slide-core.md` | frontmatter valid |
| T-05 | maxTurns 15→20 | `agents/review-core.md` | frontmatter valid |
| T-06 | `approved` field in outline.json schema + resume guard | `outline-architect.md`, `content-core.md`, `commands/ppt.md` | schema, generation, and resume logic all reference `approved` |

## Changes Summary

5 files modified, 18 insertions, 9 deletions.

### Key decisions
- Signal renamed from `draft_slide_ready(index=N)` to `draft_slides_ready(indices=[...])` to reflect batch semantics
- `approved` field defaults to `false` in generated outlines; lead sets `true` after user confirmation
- Resume logic adds a new branch: `outline.json` exists but `approved=false` → re-enter Phase 4 Hard Stop

## Next: Wave 2 — Rebuild Aesthetic Optimization Layer
- T-07+T-09 (atomic): Gemini optimizer role + suggestion taxonomy
- T-08: Technical validation fallback
- T-10/T-10b: Style palette expansion
- Calibration gate required after Wave 2
