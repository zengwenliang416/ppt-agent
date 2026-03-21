# Bento Grid Layout Specification for PPT Slides

## Overview

Bento Grid (便當網格) is a flexible card-based layout system inspired by Japanese bento boxes. Cards of varying sizes are arranged in a grid to create visual hierarchy and interest while maintaining clean organization.

## Canvas

- **Viewport**: 1280 × 720 (16:9 aspect ratio)
- **Safe Area**: 60px padding on all sides → usable area: 1160 × 600
- **Grid Base**: 12-column grid within safe area
- **Minimum Card Gap**: 20px

## Core Principles

1. **Flexibility**: Card sizes are driven by content importance, not rigid grids.
2. **Hierarchy**: Larger cards = more important content. Size communicates priority.
3. **Whitespace**: Minimum 20px gap between all cards. Whitespace is a design element, not wasted space.
4. **Balance**: Visual weight should be distributed evenly across the slide.
5. **Rhythm**: Alternate between large and small cards to create visual rhythm.

## Layout Combinations

### Single Focus
- One large card spanning the full usable area.
- Use for: title slides, key quotes, hero images, single statistics.
```
┌─────────────────────────────────┐
│                                 │
│          MAIN CONTENT           │
│                                 │
└─────────────────────────────────┘
```

### 2-Column Symmetric
- Two equal-width cards side by side.
- Use for: comparisons, before/after, pros/cons.
```
┌───────────────┐ ┌───────────────┐
│               │ │               │
│    LEFT       │ │    RIGHT      │
│               │ │               │
└───────────────┘ └───────────────┘
```

### 2-Column Asymmetric (2:1 or 1:2)
- One wide card + one narrow card.
- Use for: main content + sidebar, chart + key metrics.
```
┌─────────────────────┐ ┌───────┐
│                     │ │       │
│     MAIN (2/3)      │ │ SIDE  │
│                     │ │ (1/3) │
└─────────────────────┘ └───────┘
```

### 3-Column
- Three equal-width cards.
- Use for: three key points, triple comparison, feature grid.
```
┌─────────┐ ┌─────────┐ ┌─────────┐
│         │ │         │ │         │
│  COL 1  │ │  COL 2  │ │  COL 3  │
│         │ │         │ │         │
└─────────┘ └─────────┘ └─────────┘
```

### Hero + Grid
- One large hero card + 2-3 smaller cards.
- Use for: main point + supporting details, data overview + breakdown.
```
┌─────────────────────┐ ┌───────┐
│                     │ │ SMALL │
│     HERO (2/3)      │ ├───────┤
│                     │ │ SMALL │
└─────────────────────┘ └───────┘
```

### Mixed Grid (L-shape, T-shape, etc.)
- Various card sizes creating an irregular but balanced grid.
- Use for: dashboards, multi-metric displays, feature showcases.
```
┌─────────────────────┐ ┌───────┐
│                     │ │       │
│     LARGE           │ │ TALL  │
│                     │ │       │
├───────────┬─────────┘ │       │
│   SMALL   │  MEDIUM   │       │
└───────────┴───────────┴───────┘
```

### Timeline / Process Flow
- Horizontal flow with connected nodes for sequential content.
- Use for: project milestones, product evolution, historical timeline.
```
┌─────────────────────────────────────────────────────────┐
│  ●───────●───────●───────●───────●                      │
│  Step 1  Step 2  Step 3  Step 4  Step 5                 │
│  [desc]  [desc]  [desc]  [desc]  [desc]                 │
└─────────────────────────────────────────────────────────┘
```

### Dashboard / Mosaic Grid (4-6 cards)
- 2x2 or 2x3 grid of equal-sized metric cards.
- Use for: KPI dashboards, multi-metric overviews, feature showcases.
```
┌─────────┐ ┌─────────┐ ┌─────────┐
│  CARD 1 │ │  CARD 2 │ │  CARD 3 │
├─────────┤ ├─────────┤ ├─────────┤
│  CARD 4 │ │  CARD 5 │ │  CARD 6 │
└─────────┘ └─────────┘ └─────────┘
```

### Horizontal Split (Top + Bottom)
- Top hero card + bottom detail row.
- Use for: headline + supporting metrics, quote + evidence.
```
┌─────────────────────────────────┐
│         HEADER / HERO           │
├───────────┬─────────┬──────────┤
│  DETAIL 1 │ DETAIL 2│ DETAIL 3 │
└───────────┴─────────┴──────────┘
```

### Full-Bleed
- Edge-to-edge content with no safe area padding.
- Use for: dramatic visual slides, cover variations, section dividers.
- Override safe area: padding = 0.
```
┌─────────────────────────────────┐
│                                 │
│       FULL BLEED CONTENT        │
│                                 │
└─────────────────────────────────┘
```

## Card Aspect Ratio Constraints

- Minimum aspect ratio: 1:2 (width:height) — no card narrower than half its height
- Maximum aspect ratio: 4:1 (width:height) — no card wider than 4x its height
- Preferred ratios: 16:9, 4:3, 1:1, 3:4

## Compositional Guidelines

- For asymmetric layouts, prefer golden ratio (62:38) over arbitrary splits
- Place the most important element at rule-of-thirds intersection points:
  - Horizontal: x ≈ 427 (1/3) or x ≈ 853 (2/3)
  - Vertical: y ≈ 240 (1/3) or y ≈ 480 (2/3)
- The visual "anchor" of each slide should land near one of these 4 intersection points

## Card Anatomy

Each card contains:
- **Background**: Rounded rectangle with style-defined border-radius and optional shadow.
- **Padding**: 24px internal padding (minimum).
- **Title** (optional): Bold, larger font at top of card.
- **Content**: Text, data visualization, or icon.
- **Footer** (optional): Source, label, or secondary info.

## Typography in Cards

- **Card Title**: 24-32px, bold, primary text color.
- **Card Body**: 16-20px, regular, secondary text color.
- **Card Label**: 12-14px, uppercase or muted, tertiary text color.
- **Big Number**: 48-72px, bold, accent color (for statistics cards).

## Content-Adaptive Sizing

Adjust card internals based on content volume:

| Content Volume | Adaptation |
|---------------|------------|
| Single metric (1 number + label) | Big Number style: 48-72px centered, label below |
| Short text (< 30 words) | Standard: 16-20px body, 24px padding |
| Medium text (30-80 words) | Reduce body to 16px, padding to 20px |
| Long text (> 80 words) | Reduce body to 14px, padding to 16px |
| > 5 bullet points | Split into 2-column bullet layout within the card |
| > 8 bullet points | Split content across two cards |

When card content exceeds capacity, prefer splitting into multiple cards over shrinking text below 14px.

## Color Application

- **Card Background**: Use `card_bg` from style tokens.
- **Slide Background**: Use `background` from style tokens.
- **Accent Cards**: Use `accent` or `primary` as card background with white text for emphasis (max 1-2 per slide).

## Data Card Guidelines

When a card contains data visualization:
- **Metric cards**: Use Big Number style (48-72px number, 14-16px label below). Group 2-4 metric cards in a row.
- **Chart cards**: Minimum card width 300px for readable charts. Leave 32px padding around chart area.
- **Comparison cards**: Use horizontal bars for rankings. Align all bars to same baseline.
- **Trend cards**: Sparkline + metric in same card. Sparkline below the number, 60-80px height.

When choosing between chart and text:
- If the slide's purpose is "show the number" → Big Number card, not a chart
- If the slide's purpose is "show the trend" → Sparkline or line chart
- If the slide's purpose is "compare items" → Horizontal bar chart or comparison table
- If data has >10 entries → use a table card, not a chart

## SVG Implementation Notes

- Cards are `<rect>` elements with `rx`/`ry` for rounded corners.
- Shadows via `<filter>` with `<feDropShadow>` or `<feGaussianBlur>`.
- Text via `<text>` elements with proper `font-family`, `font-size`, `fill`.
- Use `<g transform="translate(x,y)">` to position card groups.
- All coordinates relative to `viewBox="0 0 1280 720"`.
