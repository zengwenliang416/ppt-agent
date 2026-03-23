---
name: gemini-cli
description: |
  [Trigger] When PPT workflow needs SVG slide layout and aesthetic optimization via Gemini.
  [Output] Structured optimization suggestions with scores and concrete layout/style improvements.
  [Skip] For content authoring or SVG generation tasks (those are handled by Claude).
  [Ask] No user input needed; invoked by review-core agent.
  [Resource Usage] Use references/, scripts/ (`scripts/invoke-gemini-ppt.ts`).
allowed-tools:
  - Bash
  - Read
  - Write
arguments:
  - name: role
    type: string
    required: true
    description: "reviewer"
  - name: prompt
    type: string
    required: true
    description: "Final prompt passed to Gemini"
  - name: image
    type: string
    required: false
    description: "Image path for vision tasks (rendered SVG screenshots)"
---

# Gemini CLI - PPT Slide Layout & Aesthetic Optimizer

SVG slide layout and aesthetic optimizer via `scripts/invoke-gemini-ppt.ts`. Gemini's role is to **optimize layout and improve visual aesthetics** — not just check compliance. It evaluates layout balance, visual hierarchy, color harmony, typography, and readability, then proposes concrete improvements. The script automatically tries fallback models if the primary model is unavailable.

## Script Entry

```bash
npx tsx scripts/invoke-gemini-ppt.ts \
  --role "<role>" \
  --prompt "<prompt>" \
  [--image "<path>"] \
  [--model "<model>"] \
  [--output "<path>"]
```

## Resource Usage

- Role prompts: `references/roles/{role}.md`
- Execution script: `scripts/invoke-gemini-ppt.ts`

## Roles

| Role     | Purpose                              | CLI Flag          |
| -------- | ------------------------------------ | ----------------- |
| reviewer | SVG slide layout & aesthetic optimization | `--role reviewer` |

## Workflow

### Step 1: Read the SVG source and style tokens

Before calling Gemini, read the SVG file content and the relevant style YAML so you can include them in the prompt. The reviewer needs the actual SVG source code to inspect element attributes (font-size, fill, opacity, coordinates).

### Step 2: Build the optimization prompt

Construct a prompt that includes:
- The full SVG source code (or relevant excerpts for very large files)
- The style token values (colors, fonts, border-radius, gap)
- The slide context (index, topic, presentation style name)

### Step 3: Call Gemini

```bash
npx tsx scripts/invoke-gemini-ppt.ts \
  --role reviewer \
  --prompt "$REVIEW_PROMPT" \
  --output "${RUN_DIR}/reviews/gemini-raw-${SLIDE_INDEX}.md"
```

The script tries models in order: default → gemini-2.5-pro → gemini-2.5-flash. If all fail, it exits with code 2.

### Step 4: Handle the result

- **Exit code 0**: Gemini responded. The raw output is already persisted at `${RUN_DIR}/reviews/gemini-raw-{nn}.md` — **do not delete it**. Read and extract the structured optimization suggestions.
- **Exit code 2**: All Gemini models unavailable. **Fall back to Claude self-optimization** using the same quality standards from `references/roles/reviewer.md`. This is the expected degradation path — optimization must still happen, just without the cross-model perspective.
- **Exit code 1**: Script error (bad args, missing file). Fix and retry.

### Step 5: Write the final optimization report

Whether from Gemini or Claude fallback, write the structured report to `${run_dir}/reviews/review-{nn}.md` using the output format defined in `references/roles/reviewer.md`. The Gemini raw output (`gemini-raw-{nn}.md`) is preserved as an intermediate artifact for traceability.

---

## Prompt Templates

### Role: reviewer — SVG Layout & Aesthetic Optimization

```bash
npx tsx scripts/invoke-gemini-ppt.ts \
  --role reviewer \
  --output "${RUN_DIR}/reviews/gemini-raw-${N}.md" \
  --prompt "
## Task
Optimize the SVG presentation slide's layout and visual aesthetics. Your primary output is **typed optimization suggestions** — concrete, actionable design improvements. Scoring is a secondary quality gate.

## Slide Content
${SVG_CONTENT}

## Style Reference
${STYLE_NAME} style with tokens: ${STYLE_TOKENS}

## Optimization Criteria
1. Layout Balance: card arrangement, visual weight distribution, whitespace usage
2. Color Harmony: palette consistency, contrast ratios, accent usage
3. Typography: hierarchy clarity, font size appropriateness, line spacing
4. Readability: text legibility at presentation resolution, information flow
5. Information Density: content-to-whitespace ratio, cognitive load

## Output Format
Follow the exact structure from references/roles/reviewer.md:
1. **Optimization Suggestions** (primary): typed suggestions using the 5-type taxonomy (attribute_change, layout_restructure, full_rethink, content_reduction, deck_coordination). Each with type, priority (1-3), description, and type-specific details.
2. **Suggestions JSON**: parseable JSON array of all suggestions for downstream automation.
3. **Quality Gate** (secondary): overall_score (1-10), pass (true/false), per-criterion weighted scores, hard rule violations.
"
```

---

## Fallback Strategy — Technical Validation Only

The dual-model approach provides value through independent aesthetic perspective. **Aesthetic optimization requires cross-model review** — Claude cannot meaningfully optimize its own visual design choices (production data: 12 slides all scored 8.2-8.8 with identical notes during self-review).

When Gemini is unavailable (exit code 2), review-core performs **technical validation only**:

1. Run hard-rule checks from `references/roles/reviewer.md` Quality Standards table:
   - XML validity (xmllint)
   - ViewBox present (1280x720)
   - Font-size floor (>= 14px body, >= 12px labels)
   - Safe area margins (>= 60px)
   - WCAG AA contrast ratios
   - Style token compliance (colors from declared YAML)
2. Report violations as pass/fail — **no aesthetic scores, no optimization suggestions**.
3. Mark output header as: `**Reviewer**: Claude technical validation (Gemini unavailable) — aesthetic optimization not performed`
4. The fix loop does **not** trigger for technical-only reviews (there are no typed suggestions to apply). Only hard-rule violations that are Critical or Major are reported as blocking issues.

This is an honest tradeoff: without Gemini, slides have technical correctness but no cross-model aesthetic refinement. This is better than rubber-stamp scores that give false confidence.

## Intermediate Artifact Preservation

Gemini's raw output (`gemini-raw-{nn}.md`) MUST be preserved in `${RUN_DIR}/reviews/`. These files serve as:
- **Traceability**: what Gemini actually suggested vs what was applied
- **Debugging**: if a fix round makes things worse, the original suggestion is available
- **Learning**: patterns in Gemini's suggestions can inform future style token tuning

---

## Constraints

| Required                                  | Forbidden                               |
| ----------------------------------------- | --------------------------------------- |
| MUST attempt Gemini via script first      | Skip Gemini without trying              |
| MUST fall back to technical validation on exit 2 | Fail the entire review if Gemini is down|
| MUST use reviewer role quality standards  | Send generic/empty prompts to Gemini    |
| MUST persist ALL output to run_dir artifacts | Discard Gemini output (gemini-raw-*.md) |
| Review MUST produce typed suggestions (Gemini) or hard-rule pass/fail (fallback) | Return vague qualitative-only feedback  |
| MUST include SVG source in the prompt     | Review based on filename alone          |

## Collaboration

1. **review-core** agent invokes this skill via `Skill(skill="ppt-agent:gemini-cli")` at runtime. The Skill tool loads SKILL.md into the agent's context, then the agent uses Bash to call `invoke-gemini-ppt.ts`. Both `Skill` and `Bash` must be in the agent's tools list.
2. Review output feeds back to **slide-core** for fix iterations
