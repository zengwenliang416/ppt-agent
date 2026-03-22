# Running The PPT Workflow

## Command Entry

Use `/ppt-agent:ppt` with a topic and optional flags.

Documented flags today are:

- `--style=<style>`
- `--brand-colors=<path>`
- `--pages=10-15`
- `--run-id=<id>`

The command-level argument hint lives in `commands/ppt.md`.

## What Happens During A Normal Run

1. The command creates or resumes `openspec/changes/<run_id>/`.
2. Research runs first to gather background and clarify the presentation target.
3. Materials are collected in parallel per section.
4. An outline is proposed and must be approved.
5. Draft SVGs are generated from the approved structure.
6. Final slides are designed and reviewed in a pipeline.
7. Final artifacts are copied into `output/` with preview HTML and speaker notes.

## Files To Inspect While Debugging A Run

- `input.md` - confirms parsed flags and topic.
- `requirements.md` - confirms what the user actually approved.
- `materials.md` - shows the evidence base for the outline.
- `outline.json` - the most important structural contract.
- `draft-manifest.json` - confirms draft generation coverage.
- `slide-status.json` - confirms per-slide review state and resume readiness.
- `review-manifest.json` - confirms deck-level quality status.

## Resume Behavior

When `--run-id` is supplied, the command infers where to continue by checking which artifacts already exist.

The most important operational implication is that deleting or hand-editing those checkpoint files changes resume behavior. Maintain them carefully.

## Brand Color Overrides

If `--brand-colors` is provided, the command reads a YAML file with:

```yaml
brand:
  primary: "#FF6900"
  secondary: "#000000"
  logo_text: "Mi"
```

The workflow then merges these values into `${RUN_DIR}/brand-style.yaml`.

Downstream design work should consume `brand-style.yaml` when present instead of the base style file.

## Delivery Expectations

The delivery phase is not complete until the run has:

- copied final SVGs into `${RUN_DIR}/output/`,
- generated `${RUN_DIR}/output/index.html`,
- generated `${RUN_DIR}/output/speaker-notes.md`,
- reported deck-level quality numbers sourced from `review-manifest.json`.
