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
- `${RUN_DIR}/slide-status.json`
- `${RUN_DIR}/reviews/review-{nn}.md`
- `${RUN_DIR}/reviews/review-holistic.md`
- `${RUN_DIR}/review-manifest.json`
- `${RUN_DIR}/output/`
- `${RUN_DIR}/output/index.html`
- `${RUN_DIR}/output/speaker-notes.md`

## Phase 1: Init

1. Parse flags: `--style` (optional), `--pages` (default: `10-15`), `--run-id`.
   Available styles are discovered from `skills/_shared/index.json` (filter resources where `domain == "style"`). Do NOT hardcode style names — read the registry.

   **Style selection** (if `--style` not provided, two-step flow):

   **Step 1 — Choose style group** via `AskUserQuestion` (4 options):
   | Group | Styles | Description |
   |-------|--------|-------------|
   | Professional | business, minimal, notion, scientific, editorial-infographic | 商务、学术、数据驱动场景 |
   | Creative | creative, bold-editorial, vector-illustration, sketch-notes, watercolor | 设计、营销、艺术场景 |
   | Tech / Dark | tech, blueprint, intuition-machine, pixel-art | 技术演示、产品发布、暗色系 |
   | Thematic | chalkboard, fantasy-animation, vintage | 教育、故事、复古主题 |

   **Step 2 — Choose specific style** via `AskUserQuestion` (2-4 options from the selected group):
   Read each candidate style's YAML, present with `preview` field showing:
   ```
   {name}
   Mood: {mood first sentence}
   Colors: ██ {primary} ██ {accent} ██ {card_bg}
   Font: {heading_font first entry}
   ```
   If the group has >4 styles, split into two questions or show the top 4 most relevant to the topic.

   If `--style` is explicitly provided, skip both steps.
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

   **Resume detection** (when `--run-id` is provided): Check existing artifacts to determine the resume point:
   - `slide-status.json` exists → resume Phase 6 from first incomplete slide
   - `draft-manifest.json` exists but no `slide-status.json` → resume at Phase 6 start
   - `outline.json` exists with `"approved": true` but no `draft-manifest.json` → resume at Phase 5
   - `outline.json` exists with `"approved": false` or missing → resume at Phase 4 (re-enter Hard Stop for user approval)
   - `materials.md` exists but no `outline.json` → resume at Phase 4
   - `requirements.md` exists but no `materials.md` → resume at Phase 3
   - `research-context.md` exists but no `requirements.md` → resume at Phase 2 (user confirmation)
   - Nothing exists → start from Phase 1

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
   Each agent writes to its **own isolated file** to avoid race conditions.
   Pass `research_context=research-context.md` so agents skip already-covered content and focus on incremental deep-dive:
   ```text
   Task(subagent_type="ppt-agent:research-core", name="collect-1", prompt="run_dir=${RUN_DIR} mode=collect topic=${SECTION_1} output_file=materials-${SLUG_1}.md research_context=research-context.md")
   Task(subagent_type="ppt-agent:research-core", name="collect-2", prompt="run_dir=${RUN_DIR} mode=collect topic=${SECTION_2} output_file=materials-${SLUG_2}.md research_context=research-context.md")
   Task(subagent_type="ppt-agent:research-core", name="collect-3", prompt="run_dir=${RUN_DIR} mode=collect topic=${SECTION_3} output_file=materials-${SLUG_3}.md research_context=research-context.md")
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

3. **After user approves**: Set `"approved": true` in `outline.json` and write the updated file. This field is the resume guard — `--run-id` resume checks it to determine whether Phase 4 Hard Stop was completed.

4. If user requests changes, re-run content-core in **revise** mode with the user's feedback:
   ```text
   Task(subagent_type="ppt-agent:content-core", prompt="run_dir=${RUN_DIR} mode=revise feedback=${USER_FEEDBACK}")
   ```
   content-core applies incremental changes to the existing `outline.json` (reorder, add, remove, modify) instead of regenerating from scratch. Then re-present the updated outline to the user for approval. Repeat until user approves.

## Phase 5: Planning Draft (策划稿)

1. Run draft generation:
   ```text
   Task(subagent_type="ppt-agent:content-core", prompt="run_dir=${RUN_DIR} mode=draft")
   ```
   Generates simple SVG pages: `${RUN_DIR}/drafts/slide-{nn}.svg`.
   Outputs `${RUN_DIR}/draft-manifest.json`.
   content-core sends `draft_slides_ready(indices=[N,N+1,N+2])` in batches of 3 as drafts complete, enabling Phase 6 pipeline overlap while conserving turn budget.

## Phase 6: Design Draft (设计稿) + Gemini Review

**Pipeline optimization**: Do not wait for all drafts to complete. As soon as `draft_slides_ready(indices=[...])` is received, launch slide-core for each slide in the batch. Use a sliding window of `min(3, remaining_slides)` parallel slide-core agents to balance throughput and resource usage.

**Incremental progress tracking**: Maintain `${RUN_DIR}/slide-status.json` throughout Phase 6. After each slide completes its design→review cycle (pass or accepted_with_warning), append its entry immediately. This enables `--run-id` resume to skip already-completed slides. Format:
```json
{
  "slides": {
    "01": { "status": "passed", "score": 8.2, "fix_rounds": 0, "timestamp": "2026-03-21T10:30:00Z" },
    "05": { "status": "accepted_with_warning", "score": 6.8, "fix_rounds": 2, "timestamp": "2026-03-21T10:35:00Z" }
  }
}
```
On `--run-id` resume: read `slide-status.json`, skip slides already marked `passed` or `accepted_with_warning`, and continue from the first incomplete slide.

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
   - Score < 3.0: regenerate from scratch — re-run `mode=design` **without** `fixes_json` so slide-core produces a fresh layout from the draft reference instead of patching a broken SVG.

   For fix rounds, re-run slide-core with fixes:
   ```text
   Task(subagent_type="ppt-agent:slide-core", prompt="run_dir=${RUN_DIR} mode=design slide_index=${N} style=${STYLE} fixes_json=${FIXES}")
   ```
   For regeneration (score < 3.0), re-run without fixes:
   ```text
   Task(subagent_type="ppt-agent:slide-core", prompt="run_dir=${RUN_DIR} mode=design slide_index=${N} style=${STYLE}")
   ```
   Then re-run review-core. If still failing after max rounds, accept current version with quality note.

4. **Holistic deck review** (after all individual slides pass):
   ```text
   Task(subagent_type="ppt-agent:review-core", prompt="run_dir=${RUN_DIR} mode=holistic")
   ```
   Evaluates cross-slide consistency, visual rhythm, narrative arc, and pacing.
   Holistic review produces `deck_coordination` type suggestions only. These are not auto-fixed — they are reported in `review-holistic.md` for the user to review. If holistic score < 7, flag specific issues for manual review but do not block delivery.

5. **Write review manifest** (`${RUN_DIR}/review-manifest.json`):
   After holistic review completes, the lead **aggregates from `slide-status.json`** and appends holistic results into the final checkpoint artifact. Do not re-parse individual `reviews/review-{nn}.md` files — all per-slide scores and statuses are already in `slide-status.json`:
   ```json
   {
     "total_slides": 12,
     "review_engine": "gemini" | "technical-validation-only",
     "slides": [
       {
         "index": 1,
         "file": "slides/slide-01.svg",
         "final_score": 8.2,
         "fix_rounds": 0,
         "status": "passed",
         "scores": {
           "layout": 8, "color": 9, "typography": 8, "readability": 8, "density": 8
         }
       },
       {
         "index": 5,
         "file": "slides/slide-05.svg",
         "final_score": 6.8,
         "fix_rounds": 2,
         "status": "accepted_with_warning",
         "scores": { "layout": 7, "color": 7, "typography": 7, "readability": 6, "density": 7 },
         "warning": "Readability at gate threshold after 2 fix rounds"
       }
     ],
     "holistic": {
       "score": 7.5,
       "status": "passed",
       "issues": []
     },
     "summary": {
       "passed": 10,
       "accepted_with_warning": 2,
       "avg_score": 7.8
     }
   }
   ```
   This manifest serves as the quality gate record between Phase 6 and Phase 7. Phase 7 reads it to determine which slides to deliver and to generate the quality summary.

## Phase 7: Delivery (Hard Stop)

1. **Read review manifest** (`${RUN_DIR}/review-manifest.json`) to determine delivery scope and quality status.

2. Collect all final SVGs into `${RUN_DIR}/output/`:
   ```bash
   mkdir -p "${RUN_DIR}/output"
   cp "${RUN_DIR}/slides/"*.svg "${RUN_DIR}/output/"
   ```

3. **Generate HTML preview page** (`${RUN_DIR}/output/index.html`):
   1. Read the template from `skills/_shared/assets/preview-template.html` (relative to plugin root).
   2. Read `${RUN_DIR}/outline.json` to extract slide titles for labels.
   3. Replace template placeholders:
      - `{{TITLE}}` → presentation title from outline.json (e.g. "新一代小米SU7发布会")
      - `{{LOGO}}` → short brand mark (2-3 chars, e.g. "Mi", "PPT")
      - `{{ACCENT_COLOR}}` → style accent color hex (from style YAML, e.g. `#FF6900`)
      - `{{SLIDES_JSON}}` → JSON array of slide objects:
        ```json
        [
          { "file": "slide-01.svg", "label": "封面 — 主标题" },
          { "file": "slide-02.svg", "label": "第二页标题" }
        ]
        ```
        Build labels from `outline.json`: use each page's `title` field. For the cover page use `cover.title`, for the end page use `end_page.title`.
   4. Write the populated HTML to `${RUN_DIR}/output/index.html`.

   The HTML preview supports three viewing modes:
   - **Gallery**: thumbnail grid, click any slide to present
   - **Scroll**: vertical full-width scroll through all slides
   - **Present**: fullscreen presentation with keyboard navigation

   Navigation: click left/right half of the screen or use `←→` keys to navigate slides in presentation mode.
   Keyboard shortcuts: `P` present / `G` gallery / `S` scroll / `F` fullscreen / `N` notes / `Esc` exit

4. **Generate speaker notes** (`${RUN_DIR}/output/speaker-notes.md`):
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

5. **Open preview** in the user's default browser (platform-adaptive):
   ```bash
   # macOS
   open "${RUN_DIR}/output/index.html"
   # Linux
   xdg-open "${RUN_DIR}/output/index.html"
   # Windows (WSL / Git Bash)
   start "${RUN_DIR}/output/index.html"
   ```
   Detect platform via `$OSTYPE` or `uname` and use the appropriate command.

6. Print final summary (sourced from `review-manifest.json`):
   - Total slides generated
   - Style applied
   - Review engine used (Gemini / Claude self-review)
   - Quality scores per slide: index, score, status, fix rounds
   - Slides with warnings (accepted_with_warning) highlighted
   - Holistic review score
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
- If Gemini is unavailable, review-core performs **technical validation only** — hard-rule checks (XML validity, viewBox, font-size, safe area, WCAG contrast, style tokens) with pass/fail. **No aesthetic scores and no optimization suggestions are produced.** The fix loop does not trigger for technical-only reviews. Slides pass if all Critical/Major hard rules pass. This is an honest degradation — aesthetic optimization requires cross-model review. See `skills/gemini-cli/SKILL.md` Fallback Strategy for the canonical policy.
