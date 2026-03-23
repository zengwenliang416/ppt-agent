# Changes — Wave 1: Unlock Foundation

## agents/review-core.md
- Added `Bash` to `tools` array (T-01) — unblocks 5 pre-review automated checks
- `maxTurns: 15` → `maxTurns: 20` (T-05) — sufficient for pre-checks + Gemini call + structured output

## agents/content-core.md
- `maxTurns: 25` → `maxTurns: 35` (T-02) — prevents truncation on 12+ slide decks
- Draft signaling changed from per-slide `draft_slide_ready(index=N)` to batch `draft_slides_ready(indices=[N,N+1,N+2])` every 3 slides (T-03) — conserves turn budget
- Added `approved` to required outline.json fields; content-core always generates `"approved": false` (T-06)

## agents/slide-core.md
- `maxTurns: 30` → `maxTurns: 20` (T-04) — right-sized for single-slide-per-invocation

## commands/ppt.md
- Phase 4: Added step 3 "After user approves: set approved=true in outline.json" (T-06)
- Phase 5: Updated signal description to batch semantics (T-03)
- Phase 6: Updated pipeline trigger from single index to batch indices (T-03)
- Resume detection: Added `approved` field check — `outline.json` with `approved=false` resumes at Phase 4 (T-06)

## skills/_shared/references/prompts/outline-architect.md
- Added `"approved": false` to JSON schema example (T-06)
- Added "Approval Field" section explaining semantics and resume guard contract (T-06)
