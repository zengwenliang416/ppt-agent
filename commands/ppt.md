---
description: "PPT slide generation workflow: init -> requirement research -> material collection -> outline planning -> planning draft -> design draft + Gemini review -> delivery"
argument-hint: "[--style=<style>] [--brand-colors=<path>] [--pages=10-15] [--run-id=<id>] <topic>"
allowed-tools:
  - Task
  - AskUserQuestion
  - Read
  - Write
  - Bash
---

# /ppt-agent:ppt

## Purpose

Generate professional PPT slides as SVG files (1280x720) through a structured multi-phase workflow with Bento Grid layouts, Claude draft generation, and Gemini quality review.

## Agent Types

- `ppt-agent:research-core`
- `ppt-agent:content-core`
- `ppt-agent:slide-core`
- `ppt-agent:review-core`

## Required Artifacts

- `${RUN_DIR}/input.md`
- `${RUN_DIR}/proposal.md`
- `${RUN_DIR}/tasks.md`
- `${RUN_DIR}/research-context.md`
- `${RUN_DIR}/requirements.md`
- `${RUN_DIR}/materials.md`
- `${RUN_DIR}/outline.json`
- `${RUN_DIR}/outline-preview.md`
- `${RUN_DIR}/drafts/slide-{nn}.svg`
- `${RUN_DIR}/draft-manifest.json`
- `${RUN_DIR}/slides/slide-{nn}.svg`
- `${RUN_DIR}/reviews/review-{nn}.md`
- `${RUN_DIR}/reviews/review-holistic.md`
- `${RUN_DIR}/output/`
- `${RUN_DIR}/output/index.html`
- `${RUN_DIR}/output/speaker-notes.md`

## Phase 1: Init

1. Parse flags: `--style` (default: `business`), `--pages` (default: `10-15`), `--run-id`.
   Available styles are discovered from `skills/_shared/index.json` (filter resources where `domain == "style"`). Do NOT hardcode style names — read the registry.
2. Parse `--brand-colors` flag (optional). If provided, read the brand YAML file which should contain:
   ```yaml
   brand:
     primary: "#FF6900"     # Main brand color
     secondary: "#000000"   # Secondary brand color
     logo_text: "Mi"        # Short brand mark (2-3 chars)
   ```
   Merge brand colors with the selected style: brand `primary` overrides style `accent`, brand `secondary` overrides style `primary`. The plugin derives remaining tokens (card_bg, text contrast) automatically.
   Write merged style to `${RUN_DIR}/brand-style.yaml` for downstream agents to reference.
3. Extract `<topic>` from remaining arguments.
4. Resolve run id:

   ```bash
   # If --run-id provided, resume existing run
   # Otherwise derive CHANGE_ID: kebab-case from topic
   # Examples: "ppt-ai-industry-trends", "ppt-quarterly-report"
   # Fallback: "ppt-$(date +%Y%m%d-%H%M%S)"
   if [[ -n "${RUN_ID_ARG}" ]]; then
     CHANGE_ID="${RUN_ID_ARG}"
   else
     CHANGE_ID="ppt-${slug_from_topic}"
   fi
   RUN_DIR="openspec/changes/${CHANGE_ID}"
   mkdir -p "${RUN_DIR}"
   ```

5. **Write OpenSpec scaffold** to `${RUN_DIR}/`:
   - `proposal.md`: `# Change:` title, `## Why` (presentation purpose), `## What Changes` (slide deliverables), `## Impact`
   - `tasks.md`: one numbered section per phase (Init, Research, Collection, Outline, Draft, Design, Delivery) with `- [ ]` items
   - Mark items `[x]` as each phase completes.

6. Write `${RUN_DIR}/input.md` with topic, style, page range, and flags.

## Phase 2: Requirement Research (Hard Stop)

1. Run background research:
   ```text
   Task(subagent_type="ppt-agent:research-core", prompt="run_dir=${RUN_DIR} mode=research topic=${TOPIC}")
   ```
   Agent uses `agent-reach` skill for web search and outputs `${RUN_DIR}/research-context.md`.

2. **MANDATORY**: MUST call `AskUserQuestion` to present research findings and ask:
   - Target audience (who will see this presentation?)
   - Presentation purpose (inform / persuade / teach / report?)
   - Key messages (3-5 core points to convey)
   - Tone preference (formal / casual / inspirational / data-driven?)
   - Any specific content requirements or constraints

   Do NOT proceed until user responds.

3. Write `${RUN_DIR}/requirements.md` from user answers combined with research context.

## Phase 3: Material Collection (Parallel)

1. Extract section topics from `${RUN_DIR}/requirements.md`.
2. Launch parallel collection tasks in a single message (one per major section topic).
   Each agent writes to its **own isolated file** to avoid race conditions:
   ```text
   Task(subagent_type="ppt-agent:research-core", name="collect-1", prompt="run_dir=${RUN_DIR} mode=collect topic=${SECTION_1} output_file=materials-${SLUG_1}.md")
   Task(subagent_type="ppt-agent:research-core", name="collect-2", prompt="run_dir=${RUN_DIR} mode=collect topic=${SECTION_2} output_file=materials-${SLUG_2}.md")
   Task(subagent_type="ppt-agent:research-core", name="collect-3", prompt="run_dir=${RUN_DIR} mode=collect topic=${SECTION_3} output_file=materials-${SLUG_3}.md")
   ```
3. **Lead serially merges** all per-topic files into `${RUN_DIR}/materials.md` after all agents complete.
   This avoids parallel append conflicts on a shared file.

## Phase 4: Outline Planning (Hard Stop)

1. Run outline generation:
   ```text
   Task(subagent_type="ppt-agent:content-core", prompt="run_dir=${RUN_DIR} mode=outline")
   ```
   Agent uses Pyramid Principle from `_shared/references/prompts/outline-architect.md`.
   Outputs `${RUN_DIR}/outline.json` and `${RUN_DIR}/outline-preview.md`.

2. **MANDATORY**: MUST call `AskUserQuestion` to present the outline as "digital sticky notes" view.
   User can approve, adjust page order, add/remove sections, or change emphasis.
   Do NOT proceed until user approves.

3. If user requests changes, re-run content-core with adjusted requirements.

## Phase 5: Planning Draft (策劃稿)

1. Run draft generation:
   ```text
   Task(subagent_type="ppt-agent:content-core", prompt="run_dir=${RUN_DIR} mode=draft")
   ```
   Generates simple SVG pages: `${RUN_DIR}/drafts/slide-{nn}.svg`.
   Outputs `${RUN_DIR}/draft-manifest.json`.
   content-core sends `draft_slide_ready(index=N)` per-slide as each draft completes, enabling Phase 6 to begin designing earlier slides while later drafts are still being generated.

## Phase 6: Design Draft (設計稿) + Gemini Review

**Pipeline optimization**: Do not wait for all drafts to complete. As soon as `draft_slide_ready(index=N)` is received, launch slide-core for that slide. Use a sliding window of `min(3, remaining_slides)` parallel slide-core agents to balance throughput and resource usage.

For each slide (or in batches):

1. **Claude generates** final design SVG:
   ```text
   Task(subagent_type="ppt-agent:slide-core", prompt="run_dir=${RUN_DIR} mode=design slide_index=${N} style=${STYLE}")
   ```
   Applies Bento Grid layout + style tokens → `${RUN_DIR}/slides/slide-{nn}.svg`.
   slide-core performs post-generation automated validation (XML validity, viewBox, font-size floor, safe area) before reporting completion. review-core also runs pre-review automated checks to skip expensive LLM review for trivially broken SVGs.

2. **Gemini reviews** the generated SVG:
   ```text
   Task(subagent_type="ppt-agent:review-core", prompt="run_dir=${RUN_DIR} mode=review slide_index=${N}")
   ```
   Checks layout balance, color harmony, typography, readability, information density.
   Outputs `${RUN_DIR}/reviews/review-{nn}.md` with pass/fail + fix suggestions.

3. **Fix loop** (adaptive budget per Weighted Scoring Model): if review fails, action depends on initial weighted score:
   - Score >= 7.0 + gates pass: no fixes needed.
   - Score 5.0–6.9: fix loop, max 2 rounds.
   - Score 3.0–4.9: fix loop, max 1 round (unlikely to reach 7 in 2).
   - Score < 3.0: regenerate from scratch (`mode=regenerate`) — do not patch.

   For fix rounds, re-run slide-core with fixes:
   ```text
   Task(subagent_type="ppt-agent:slide-core", prompt="run_dir=${RUN_DIR} mode=design slide_index=${N} style=${STYLE} fixes_json=${FIXES}")
   ```
   Then re-run review-core. If still failing after max rounds, accept current version with quality note.

4. **Holistic deck review** (after all individual slides pass):
   ```text
   Task(subagent_type="ppt-agent:review-core", prompt="run_dir=${RUN_DIR} mode=holistic")
   ```
   Evaluates cross-slide consistency, visual rhythm, narrative arc, and pacing.
   If holistic score < 7, flag specific issues for manual review but do not block delivery.

## Phase 7: Delivery (Hard Stop)

1. Collect all final SVGs into `${RUN_DIR}/output/`:
   ```bash
   mkdir -p "${RUN_DIR}/output"
   cp "${RUN_DIR}/slides/"*.svg "${RUN_DIR}/output/"
   ```

2. **Generate HTML preview page** (`${RUN_DIR}/output/index.html`):
   1. Read the template from `plugins/ppt-agent/skills/_shared/assets/preview-template.html`.
   2. Read `${RUN_DIR}/outline.json` to extract slide titles for labels.
   3. Replace template placeholders:
      - `{{TITLE}}` → presentation title from outline.json (e.g. "新一代小米SU7發布會")
      - `{{LOGO}}` → short brand mark (2-3 chars, e.g. "Mi", "PPT")
      - `{{ACCENT_COLOR}}` → style accent color hex (from style YAML, e.g. `#FF6900`)
      - `{{SLIDES_JSON}}` → JSON array of slide objects:
        ```json
        [
          { "file": "slide-01.svg", "label": "封面 — 主標題" },
          { "file": "slide-02.svg", "label": "第二頁標題" }
        ]
        ```
        Build labels from `outline.json`: use each page's `title` field. For the cover page use `cover.title`, for the end page use `end_page.title`.
   4. Write the populated HTML to `${RUN_DIR}/output/index.html`.

   The HTML preview supports three viewing modes:
   - **Gallery**: thumbnail grid, click any slide to present
   - **Scroll**: vertical full-width scroll through all slides
   - **Present**: fullscreen presentation with keyboard navigation

   Keyboard shortcuts: `P` present / `G` gallery / `S` scroll / `F` fullscreen / `N` notes / `←→` navigate / `Esc` exit

3. **Generate speaker notes** (`${RUN_DIR}/output/speaker-notes.md`):
   Read `${RUN_DIR}/outline.json` and extract all `notes` fields. Format as a markdown document indexed by slide number:
   ```markdown
   # Speaker Notes: {presentation_title}

   ## Slide 01: {title}
   **Talking Points:**
   - Point 1
   - Point 2

   **Transition:** "Now let's look at..."
   **Time:** ~2 minutes

   ---
   ## Slide 02: {title}
   ...
   ```
   Include total estimated presentation time at the bottom.

4. **Open preview** in the user's default browser (platform-adaptive):
   ```bash
   # macOS
   open "${RUN_DIR}/output/index.html"
   # Linux
   xdg-open "${RUN_DIR}/output/index.html"
   # Windows (WSL / Git Bash)
   start "${RUN_DIR}/output/index.html"
   ```
   Detect platform via `$OSTYPE` or `uname` and use the appropriate command.

5. Print final summary:
   - Total slides generated
   - Style applied
   - Quality scores per slide (from review files)
   - Average quality score
   - File paths to output SVGs
   - Preview URL: `${RUN_DIR}/output/index.html`
   - Resume command: `/ppt-agent:ppt --run-id=${CHANGE_ID}`

## Quality Gates

- Weighted score >= 7.0 per slide with hard gates: Layout >= 6, Readability >= 6
- No Critical issues (text overflow, broken layout)
- Adaptive fix budget: score >= 7 pass, 5-6.9 fix (2 rounds max), 3-4.9 fix (1 round), <3 regenerate
- All SVGs must be valid 1280x720
- Holistic deck coherence score >= 7 (advisory, does not block delivery)

## Fallback Rules

- If research fails, continue with user-provided requirements only and mark confidence downgrade.
- If material collection partially fails, continue with available materials.
- If a slide review fails after 2 fix rounds, accept with quality warning in delivery summary.
- If Gemini is unavailable, review-core falls back to Claude self-review using the same quality standards from `gemini-cli/references/roles/reviewer.md`. The review MUST still happen — only the cross-model perspective is lost. See `skills/gemini-cli/SKILL.md` Fallback Strategy for the canonical policy.
