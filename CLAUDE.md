# PPT Agent Plugin

Always answer in Chinese (Traditional).

<available-skills>

| Skill             | Trigger                                                                   | Description                                                              |
| ----------------- | ------------------------------------------------------------------------- | ------------------------------------------------------------------------ |
| `/ppt-agent:ppt`  | "PPT", "slides", "幻燈片", "做個PPT", "slide deck", "簡報", "做簡報" | PPT slide generation workflow: init → research → outline → design → delivery |

</available-skills>

## Available Commands

- `/ppt-agent:ppt`: End-to-end PPT slide generation workflow.

## Workflow Phases

1. Init: Parse arguments, create `openspec/changes/<run_id>/`.
2. Requirement Research (Hard Stop): Background search + user clarification.
3. Material Collection (Parallel): Per-section deep search.
4. Outline Planning (Hard Stop): Pyramid Principle outline + user approval.
5. Planning Draft: Simple SVG pages per slide (per-slide signaling enables Phase 6 pipeline).
6. Design Draft + Gemini Review: Bento Grid SVG generation + quality review loop.
7. Delivery (Hard Stop): Collect final SVGs + HTML preview + summary.

## Agent Types

- `ppt-agent:research-core` — requirement research + material collection
- `ppt-agent:content-core` — outline planning + planning draft
- `ppt-agent:slide-core` — design SVG generation with Bento Grid
- `ppt-agent:review-core` — Gemini-powered SVG quality review

All agents are spawned via `Task()` calls. No Agent Team required — agents communicate only with the lead orchestrator via `SendMessage` (heartbeat, ready, error signals), not with each other directly.

## Skill Invocation Conventions

- `ppt-agent:research-core` uses `agent-reach` skill for web search.
- `ppt-agent:review-core` invokes `ppt-agent:gemini-cli` for SVG review.
- Prompt references are in `skills/_shared/references/prompts/`.
- Style tokens are in `skills/_shared/references/styles/`. Available styles are discovered from `skills/_shared/index.json` (domain=style).
- HTML preview template is in `skills/_shared/assets/preview-template.html`.
- `outline.json` schema is defined in `skills/_shared/references/prompts/outline-architect.md` (single source of truth).
- Review fallback policy is defined in `skills/gemini-cli/SKILL.md` (single source of truth). When Gemini is unavailable, Claude self-review is used — reviews are never skipped.
- Brand color override is supported via `--brand-colors=<path>` flag. Brand styles are written to `${RUN_DIR}/brand-style.yaml`.

## HTML Preview Template

The delivery phase generates an interactive `index.html` alongside the SVG files. The template uses 4 placeholders:

| Placeholder | Source | Example |
|-------------|--------|---------|
| `{{TITLE}}` | `outline.json` title | `新一代小米SU7發布會` |
| `{{LOGO}}` | Short brand mark (2-3 chars) | `Mi`, `PPT` |
| `{{ACCENT_COLOR}}` | Style YAML accent hex | `#FF6900` |
| `{{SLIDES_JSON}}` | JSON array from outline.json | `[{"file":"slide-01.svg","label":"封面"}]` |

Preview modes: Gallery (thumbnail grid) / Scroll (vertical) / Present (fullscreen).
Keyboard: `P` present / `G` gallery / `S` scroll / `F` fullscreen / `←→` navigate / `Esc` exit.

## Quality Gates

- Layout score >= 7/10
- No critical review issues
- Maximum 2 fix rounds per slide
- All slides must be valid SVG 1280x720

## Output Directory

- `research-context.md`
- `requirements.md`
- `materials.md`
- `outline.json` + `outline-preview.md`
- `drafts/slide-{nn}.svg`
- `draft-manifest.json`
- `slides/slide-{nn}.svg`
- `reviews/review-{nn}.md`
- `output/` — final deliverables:
  - `slide-{nn}.svg` — final design SVGs
  - `index.html` — interactive HTML preview page
