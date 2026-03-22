# Updating Styles And Review

## Style Registry

Available styles are discovered from `skills/_shared/index.json`, not from hardcoded lists.

At the time of writing, the registry exposes four styles:

- `business`
- `tech`
- `creative`
- `minimal`

If a style is added or renamed, update the registry and the corresponding YAML file together.

## Style Token Model

Style YAML files define more than color and typography. The current token families include:

- `color_scheme`
- `typography`
- `card_style`
- `gradients`
- `elevation`
- `decoration`
- `slide_type_overrides`

These extended v1.1 tokens matter because the slide and review systems are both aware of them.

## Bento Grid Constraints

The layout contract comes from `skills/_shared/references/prompts/bento-grid-layout.md`.

Important constants:

- viewport: `1280x720`
- safe area: `60px`
- minimum card gap: `20px`
- twelve-column base grid inside the safe area

The allowed layout types are:

- `single_focus`
- `two_column_symmetric`
- `two_column_asymmetric`
- `three_column`
- `hero_grid`
- `mixed_grid`
- `timeline`
- `dashboard`
- `horizontal_split`
- `full_bleed`

When documenting or changing layout behavior, keep names identical to the content contract used in `outline.json`.

## Review Model

`review-core` evaluates slides against deterministic checks and then a structured design review.

The scoring model defined in `skills/gemini-cli/references/roles/reviewer.md` uses weighted criteria:

- Layout Balance 25%
- Readability 25%
- Typography 20%
- Information Density 20%
- Color Harmony 10%

Pass requires all of the following:

- overall weighted score `>= 7.0`
- Layout Balance `>= 6`
- Readability `>= 6`
- no Critical issues

## Fix Loop Semantics

- score `>= 7.0` and gates pass -> accept directly
- score `5.0-6.9` -> up to 2 fix rounds
- score `3.0-4.9` -> up to 1 fix round
- score `< 3.0` -> regenerate from scratch instead of patching

This policy is what makes review output operational instead of purely descriptive.

## Gemini Fallback

The review path must attempt Gemini first through `skills/gemini-cli/scripts/invoke-gemini-ppt.ts`.

If Gemini is unavailable, the system falls back to Claude self-review using the same reviewer contract. Review is never skipped just because cross-model optimization is unavailable.
