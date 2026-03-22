# Research And Material Collection

## Purpose

The research subsystem exists to turn a vague presentation request into a sourced, presentation-ready evidence base.

It is centered on `agents/research-core.md` and `skills/agent-reach/SKILL.md`.

## Two Distinct Modes

### Requirement Research

`mode=research` is broad and exploratory.

Its output is `${RUN_DIR}/research-context.md`, which should summarize:

- background,
- key insights,
- common presentation angles,
- suggested focus areas.

### Material Collection

`mode=collect` is narrower and section-oriented.

It reads previously generated context when available, focuses on incremental detail, and writes to an isolated `materials-*.md` file for later merging.

## Tool Selection Strategy

`research-core` probes available upstream tools once per session by running:

`bash skills/agent-reach/scripts/probe.sh`

Then it chooses the best available path, degrading from richer platform-specific options toward generic web search.

This means the research contract is environment-adaptive rather than assuming a fixed toolchain.

## Why Isolated Collection Files Matter

Per-topic collection tasks do not append to a shared `materials.md` file. They write isolated files and let the lead merge them later.

That design keeps parallel collection deterministic and avoids clobbering data during concurrent writes.

## Maintainer Checklist

When updating research behavior, verify these questions:

1. Does the probe-first flow still reflect the actual tools the skill can use?
2. Does fallback behavior still end at a viable minimum path?
3. Are outputs still separated into `research-context.md` and isolated `materials-*.md` files?
4. Are the required user questions in Phase 2 still aligned with the shape of `requirements.md`?

## Where To Extend

- Add new retrieval capabilities in `skills/agent-reach/SKILL.md` and `scripts/probe.sh`.
- Update `agents/research-core.md` when the routing or output contract changes.
- Update `commands/ppt.md` if phase sequencing, required questions, or merge behavior changes.
