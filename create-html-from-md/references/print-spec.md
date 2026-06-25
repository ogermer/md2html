# Printable HTML Slides — Briefing & Pattern Library

**Version:** 1.1
**Date:** 2026-05-27

**Purpose.** A reusable briefing for generating multi-page, print-ready HTML documents — brand books, pitch decks, handbooks, reports — that produce a clean A4 PDF via the browser's print dialog. Captures the page geometry, slide types, pagination rules, and pitfalls developed for the reference brand guide. Brand-agnostic where possible; swap in colours, type, and content for any project.

**Canonical example.** The reference brand-guide print HTML this briefing was extracted from — its screen sibling and Markdown source are the visual reference for the patterns below.

**Format.** A4 landscape, 297 × 210 mm. The recipes below assume landscape. Portrait is mechanically the same with width/height swapped.

**What changed in 1.1.** Added §6.5 — box weight & elevation. Card-class borders run ~30% heavier than the original baseline, and every card carries a subtle two-stop drop shadow that survives the print dialog (assuming Background graphics = ON, already mandated in §11).

---

## 1. Why HTML for print at all

HTML+CSS is good enough for a printed PDF when the document is:

- a few dozen to a couple of hundred pages,
- not destined for offset printing (no CMYK, no bleed marks, no spot colour),
- needing to evolve in lockstep with a screen edition that shares the same content,
- being version-controlled and produced by code rather than InDesign.

When *any* of those is false — large print runs, bound books, precise typographic control — use InDesign or Affinity Publisher. Otherwise the HTML→PDF path beats them on iteration speed, accessibility, version control, and reuse.

---

## 2. Core principles — non-negotiable

These are the invariants. Everything else in this briefing rests on them.

1. **`@page { size: A4 landscape; margin: 0 }`.** This makes the printable area the full physical page. Without it the browser inserts default 10mm-ish margins regardless of CSS.
2. **Each printed page is one `.page` element** with `width: 297mm` and `height: 209mm` (**one mm less than A4**'s 210mm). The -1mm trick prevents subpixel overflow that otherwise emits blank trailing pages in Chrome.
3. **Use only modern `break-after: page`.** Never combine with the legacy `page-break-after: always` — Chrome and Safari treat the combination as a double break and emit a blank page after every page.
4. **Wrap print-specific rules in `@media print`** and use `!important` for `margin`, `padding`, and `box-shadow`. Screen-only styles must not leak in.
5. **`page-break-inside: avoid` + `break-inside: avoid`** on every page and every block that must never split.
6. **`overflow: hidden` as a defensive backstop** on `.page` so any miscalculation clips silently rather than corrupting the page count.
7. **Document the print-dialog requirements loudly** in an on-screen banner: **Margins → None**, **Background graphics → on**, **Scale → 100%**, headers/footers off. The `@page` margin is only respected when the dialog says None.
8. **Do not use CSS multi-column for print.** `columns: 2` collapses to one column in Chrome when the parent has a fixed height. Compute splits at generation time and emit two or three `<div class="col">` elements inside a flexbox row.

---

## 3. Page geometry — A4 landscape

```
┌──────────────────────────────────────────────────┐  ← 0mm
│   pad-t = 11 mm                                  │
│ ┌──────────────────────────────────────────────┐ │
│ │  HEADER BAND   (chip · brand)                │ │  ~9mm
│ ├──────────────────────────────────────────────┤ │
│ │                                              │ │
│ │  TITLE  (h1, action title)                   │ │
│ │  ─────  (accent bar)                         │ │
│ │  subtitle                                    │ │
│ │                                              │ │
│ │  BODY (12-column flex grid; 1, 2 or 3 cols)  │ │  ~140mm
│ │                                              │ │
│ │  ──── bottom-callout ────                    │ │
│ ├──────────────────────────────────────────────┤ │
│ │  FOOTER BAND   (section title · page)        │ │  ~6mm
│ └──────────────────────────────────────────────┘ │
│   pad-b = 10 mm                                  │
└──────────────────────────────────────────────────┘  ← 209mm
←─────────────  pad-x = 14mm  ────────────────────→
       297mm total width
```

**Budget arithmetic.** Page is 297 × 209 mm. Padding takes 14mm on each side and 11mm/10mm top/bottom, leaving a usable inner area of **269 × 188 mm**. Within that, header (~9mm + 5mm rule + 3mm padding) and footer (~6mm + 4mm padding) eat another ~27mm vertical, leaving **~161 mm for the actual body**. Plan content to fit comfortably inside that.

**Full-bleed pages** (cover, part dividers, colophon) use a larger padding — `16mm 18mm 14mm` — and need no separator rules; the dark background is the structure.

---

## 4. The .page element — the recurring unit

Every printed page is one `.page` element. It is the single most important abstraction in this template.

```html
<section class="page">
  <header class="page-head">
    <span>01 · Section name</span>
    <span class="brand">BrandName</span>
  </header>
  <div class="page-body">
    <h1>Action title.</h1>
    <div class="accent-bar"></div>
    <p class="subtitle">A short subtitle, restating what the page argues.</p>

    <!-- body content here -->

    <div class="bottom-callout"><em>One italic sentence that compresses the page.</em></div>
  </div>
  <footer class="page-foot">
    <span>01 · Section name</span><span>Page N</span>
  </footer>
</section>
```

Key CSS:

```css
.page {
  width: 297mm;
  height: 209mm;            /* the -1mm trick */
  overflow: hidden;         /* defensive backstop */
  background: #FFF;
  position: relative;
  padding: 11mm 14mm 10mm;
  display: flex;
  flex-direction: column;
  page-break-inside: avoid;
  break-inside: avoid;
  break-after: page;        /* modern only */
}
.page:last-child { break-after: auto; }
.page-body { flex: 1 1 auto; display: flex; flex-direction: column; overflow: hidden; min-height: 0; }
```

The `page-body` is a flex column. This lets the **bottom callout** stick to the bottom of the body region via `margin-top: auto`, regardless of how much content sits above it. That single trick is what gives every page the same closing-italic rhythm.

---

## 5. Slide types — the catalogue

Eleven recurring layouts. Each entry below lists when to use it, the anatomy, an HTML skeleton, and the capacity (how much content fits comfortably on one A4-landscape page).

### 5.1 Cover

Use as page 1, sometimes also as a back-cover/colophon. Full-bleed dark colour, large title, lede, and a meta strip at the bottom.

- Full-bleed background colour
- A small "tag pill" above the title
- A massive title (~70pt)
- A lede paragraph (~14pt, slightly muted)
- A meta row at the bottom: four columns of label/value

```html
<section class="page cover">
  <header class="page-head"><span>Brand · Document type</span><span>Edition · Date</span></header>
  <div class="page-body">
    <div class="cover-center">
      <span class="tag">A short positioning line</span>
      <h1>Big<br>Title.</h1>
      <p class="lede">One paragraph that frames the document.</p>
    </div>
    <div class="cover-meta-row">
      <div>Document<span>Name</span></div>
      <div>Edition<span>1.0 · 2026-05-15</span></div>
      <div>Owner<span>Brand · email</span></div>
      <div>Source<span>file paths or upstream docs</span></div>
    </div>
  </div>
  <footer class="page-foot"><span>Print edition · A4 landscape</span><span>Cover</span></footer>
</section>
```

Capacity: keep the title ≤ 2 lines, lede ≤ 4 lines.

### 5.2 Part divider

Use between chapter-sized sections of a long document. Full-bleed gradient or solid, a small "Part N" eyebrow, large title, short lede. Vertically centred.

```html
<section class="page part-divider">
  <header class="page-head"><span>Part II</span><span class="brand">BrandName</span></header>
  <div class="page-body">
    <div class="part-number">Part II</div>
    <h1>Section<br>Name.</h1>
    <p class="part-lede">A one-paragraph orientation for what this part covers.</p>
  </div>
  <footer class="page-foot"><span>Part II — Section</span><span>Page N</span></footer>
</section>
```

Capacity: lede ≤ 3 lines. The page should feel almost empty — that's the point.

### 5.3 Standard spread (single-column body)

The default content page. Action title + accent bar + subtitle + body + bottom callout.

```html
<section class="page">
  <header class="page-head"><span>NN · Section</span><span class="brand">BrandName</span></header>
  <div class="page-body">
    <h1>Action title — a claim, not a topic.</h1>
    <div class="accent-bar"></div>
    <p class="subtitle">One-sentence subtitle.</p>

    <p>Body paragraphs in Slate 600.</p>
    <p>A second paragraph if needed. Aim for 3–6 short paragraphs maximum on a single-column page.</p>

    <div class="bottom-callout"><em>The one-line italic compression of this page's argument.</em></div>
  </div>
  <footer class="page-foot"><span>NN · Section</span><span>Page N</span></footer>
</section>
```

Capacity: ~400–600 words of prose at 10.5pt, line-height 1.45.

### 5.4 Two-column body

For most content-rich pages. Two columns balance prose density and give every page a visual rhythm.

```html
<div class="cols">
  <div class="col"> ... left column ... </div>
  <div class="col"> ... right column ... </div>
</div>
```

CSS:

```css
.cols { display: flex; gap: 8mm; align-items: flex-start; }
.cols .col { flex: 1 1 0; min-width: 0; }
.cols .col > * { break-inside: avoid; }
```

Capacity per column: ~250 words at 10.5pt. Two columns ≈ 500–600 words total — same as a single-column page but feels more structured. If the content is a long enumerated list, split it across the two columns by weight, not by item count.

### 5.5 Three-column body

For dense reference pages — table of contents, long lists, glossaries. Three columns at landscape width give ~84mm per column.

```html
<section class="page toc-page">
  <header class="page-head">...</header>
  <style>
    .toc-page .col h3 { font-size: 8.5pt; letter-spacing: 0.08em; text-transform: uppercase; }
    .toc-page .col ol { margin: 0 0 0.8mm; }
    .toc-page .col ol li { font-size: 8.8pt; line-height: 1.32; margin: 0.3mm 0; padding-left: 6.5mm; }
  </style>
  <div class="page-body">
    <h1>Contents.</h1>
    <div class="accent-bar"></div>
    <div class="cols">
      <div class="col"> ... ~20 items ... </div>
      <div class="col"> ... ~16 items ... </div>
      <div class="col"> ... ~22 items ... </div>
    </div>
  </div>
  <footer class="page-foot">...</footer>
</section>
```

**Balance by weight, not count.** Group items so each column's total visual height is roughly equal. For a TOC with headed sections of varying length, find the split that minimises `max(col_heights)`.

**Tighten spacing.** A three-column page often needs smaller font (8.5–9pt instead of 10pt) and smaller line height (1.32 instead of 1.45). Use scoped overrides under a parent class so you don't disturb the rest of the document.

Capacity: ~60 short list items + 4–6 sub-headers across three columns.

### 5.6 Centred statement page

For mission, vision, positioning statements, or any "single big quote" page. Vertically centred large italicised blockquote.

```html
<div class="page-body">
  <h1>Mission.</h1>
  <div class="accent-bar"></div>
  <div style="display:flex;flex-direction:column;justify-content:center;flex:1;">
    <blockquote style="font-size:18pt;line-height:1.4;padding:8mm 10mm;">
      <em>One sentence, sometimes two, in 18pt italic.</em>
    </blockquote>
  </div>
  <div class="bottom-callout teal"><em>One-line italic compression.</em></div>
</div>
```

Capacity: 1–2 sentences. The page must feel deliberate, not sparse.

### 5.7 Component-pattern pages

Pages that *demonstrate* a content pattern by showing a live instance of it. The pattern library used in the {{COMPANY_NAME}} brand book:

- **Three-stat** — three cards on a row, each with a large stat in tabular figures, a short label, and a source.
- **Three-windows** — three colour-coded cards (Navy / Amber / Teal), each with a heading, body, and date.
- **Compare-and-contrast** — two cards (red-tinted "without"; green-tinted "with") plus a centred cycle-time callout.
- **Four-pillars** — four equal columns each with a dark header band and 3 bullets.
- **Big-date callout + mapping** — an amber/yellow date card on the left, a two-column mapping table on the right with arrow connectors.
- **Quote grid** — 2×2 grid of quote cards, each with a coloured left bar, large quotation glyph, quote text, and attribution.
- **Swatch grid** — 4-up grid of colour cards, each with a tall colour chip, name, and hex.
- **Type specimen** — left column with type metadata, right column with rendered sample at actual size.
- **Logo specimen** — large rendered wordmark on a white card; mark variants in a separate "mark row" with size labels.
- **Glance grid** — 2-card overview row used at the start of "brand at a glance" / "key facts" pages.
- **Do / Don't grid** — two cards side-by-side, one with a red left bar, one with a green left bar, each carrying an italic example.

Skeleton for any pattern page: same standard spread as §5.3 but with the pattern component in the body. Capacity depends on the pattern — most fit comfortably with a short intro paragraph above and a bottom callout below.

### 5.8 Front-matter pages

Pages that explain how to use the document. Standard spreads with descriptive content rather than argumentative content. Examples: "How to use", "Contents", "Acceptance checklist", "Changelog".

### 5.9 Colophon

End-of-book version of the cover. Same `.page.cover` class. Uses softer copy ("End of book") and replaces the lede with a short retrospective.

---

## 6. Print bleed for full-colour pages

The -1mm trick (§2.2) means each `.page` is 1mm shorter than the physical page. On full-colour pages (cover, part dividers, colophon) this 1mm gap is a visible white strip at the bottom. To hide it — and to absorb any small dialog-margin errors when users forget to set Margins → None — add **5mm of print bleed** in all directions on coloured pages.

For solid backgrounds, use a 5mm `box-shadow` spread:

```css
@media print {
  .page.cover {
    box-shadow: 0 0 0 5mm var(--brand-darkest) !important;
  }
}
```

Box-shadow is drawn outside the box's overflow area, so it works cleanly with the existing `overflow: hidden`.

For gradient backgrounds, `box-shadow` can't carry a gradient. Use an absolutely-positioned `::before` with `inset: -5mm`, and override `overflow` only on that page:

```css
@media print {
  .page.part-divider {
    overflow: visible !important;
    position: relative;
  }
  .page.part-divider::before {
    content: '';
    position: absolute;
    inset: -5mm;
    background: linear-gradient(135deg, var(--brand-darkest) 0%, var(--brand-mid) 100%);
    z-index: -1;
  }
}
```

5mm is enough to cover the 1mm subpixel gap **and** absorb minor dialog-margin slop. Larger bleeds (10–15mm) are wasteful unless the document is destined for a physical printer with actual bleed cropping.

---

## 6.5 Box weight & elevation

Printed A4 landscape pages are read at arm's length; a hairline at the screen-default `0.2mm` reads thin on paper. Two adjustments together — heavier borders plus a subtle drop shadow — make every card-class element carry the visual weight it earned, without crossing into Material-card territory.

### 6.5.1 Border weights

Multiply the original screen-baseline values by **1.3** for the print edition. The screen edition keeps the lighter baseline; the print edition needs the bump.

| Role | Original | Print (×1.3) |
|------|----------|--------------|
| Hairline rule (card outline, header rule, table row) | `0.2mm` | **`0.26mm`** |
| Top accent (req-card, layer rule) | `0.6mm` | **`0.78mm`** |
| Standard accent bar (stat-card top, buyer-card top, callout left) | `0.7mm` | **`0.91mm`** |
| Strong accent (compare-card emphasis, four-pillar head divider) | `0.8mm` | **`1.04mm`** |
| Maximal accent (rare — {{COMPANY_NAME}}-lane mid emphasis) | `1.1mm` | **`1.43mm`** |

The scale-up never touches `border-radius` (corner rounding stays 0.6–0.8mm per pattern), `border-collapse`, or `border-spacing` — those are layout primitives, not visual weight.

### 6.5.2 Drop shadow on every card class

A single rule covers every card-class element. The two-stop pattern (tight ambient + soft drop) gives a CSS-elevation-token feel at low opacity:

```css
.stat-card,
.window-card,
.compare-card,
.pillar,
.arch-stage,
.layer,
.flywheel-step,
.buyer-card,
.discovery-card,
.level-step,
.req-card,
.lane-row,
.date-card,
.mapping,
.band-card {
  box-shadow: 0 0.25mm 0.7mm rgba(15, 20, 34, 0.08),
              0 0.1mm 0.25mm rgba(15, 20, 34, 0.06);
}

@media print {
  /* Many print engines drop shadows by default. Force them on. */
  .stat-card, .window-card, .compare-card, .pillar, .arch-stage,
  .layer, .flywheel-step, .buyer-card, .discovery-card, .level-step,
  .req-card, .lane-row, .date-card, .mapping, .band-card {
    -webkit-print-color-adjust: exact !important;
    print-color-adjust: exact !important;
  }
}
```

**The `print-color-adjust: exact` bit is non-optional.** Chrome and Safari otherwise treat box shadows as decorative and may strip them when the print dialog runs unless Background graphics is on AND the print-color-adjust property is set. Belt and braces.

**Opacities (0.06–0.08) are chosen so the shadow registers at presenter / reader distance but disappears at desk-reading distance** — exactly where the eye expects an elevated card without it screaming "Material Design".

### 6.5.3 What does not get a shadow

- `.bottom-callout`, `.compare-callout`, `.cycle-callout`, any callout-bar element — flat by design; the colour does the work.
- The `.page` itself — the page IS the surface, not a card on a surface.
- Cover and part-divider full-bleed colour fields — the bleed is the structure.

### 6.5.4 Author requirement

When a new card-class pattern is added (e.g. a new pricing card), **append its class to both selector lists** (the box-shadow rule and the `print-color-adjust` rule). The simplest way to keep the catalogue consistent is to grep for `box-shadow:` and confirm every card class appears.

---

## 7. Header and footer bands

Every `.page` carries a thin header band and a thin footer band, separated from the body by a hairline rule. These bands are the navigation aid — section chip on the left, brand mark on the right, page number in the corner.

```css
.page-head {
  display: flex; justify-content: space-between; align-items: center;
  font-size: 8pt;
  letter-spacing: 0.08em;
  text-transform: uppercase;
  color: var(--text-muted);
  font-variant-numeric: tabular-nums;
  margin-bottom: 5mm;
  font-weight: 500;
  border-bottom: 0.3mm solid var(--rule);
  padding-bottom: 3mm;
}
.page-foot {
  /* mirror of page-head, with border-top instead of border-bottom */
}
```

On full-bleed coloured pages, override the band colours to a light tint of the background so the rule and text remain legible.

---

## 8. Typography for print at A4-landscape

The {{COMPANY_NAME}} book uses two families: **Inter** for everything (display, UI, body, captions, numerics) and **Source Serif 4** for editorial long-form. Both are loaded from Google Fonts; the system fallback chain is identical to the screen edition.

Scale used at A4-landscape (pt, since print sizing in mm is awkward):

| Role                         | Size        | Weight | Line height | Tracking      |
|------------------------------|-------------|--------|-------------|---------------|
| Cover h1                     | 70pt        | 700    | 1.0         | -0.035em      |
| Part divider h1              | 56pt        | 700    | 1.0         | -0.03em       |
| Page h1 (action title)       | 26pt        | 700    | 1.1         | -0.015em      |
| Section h2                   | 13pt        | 600    | 1.25        | -0.005em      |
| Sub-section h3               | 11pt        | 600    | 1.3         | 0             |
| Caption / metadata           | 7.5–8pt     | 500    | 1.4         | 0.08em UPPER  |
| Subtitle (lede)              | 12.5pt      | 400    | 1.4         | 0             |
| Body                         | 10.5pt      | 400    | 1.45        | 0             |
| Body small (table cells)     | 9pt         | 400    | 1.4         | 0             |
| Big number (stat card)       | 32pt        | 700    | 1.0         | -0.03em       |
| Display sample (type page)   | 26pt        | 600    | 1.05        | -0.025em      |

**Tabular figures.** Any column of numbers — stat cards, dates, table values, page numbers — uses `font-variant-numeric: tabular-nums` so digit widths align. Proportional figures elsewhere.

---

## 9. Capacity & overflow — how much fits, and what to do when it doesn't

For a single A4-landscape page with the geometry in §3, plan for:

| Layout                  | Comfortable capacity                              |
|-------------------------|---------------------------------------------------|
| Single column prose     | ~400–600 words at 10.5pt                          |
| Two-column prose        | ~500–600 words total (~250 per column)            |
| Three-column reference  | ~60 list items + 4–6 sub-headers                  |
| Stat / pattern card     | 3–5 bullets per card                              |
| Table                   | 8–12 rows at standard padding (1.1mm cell)        |
| Code block              | 18–22 lines at 8.5pt monospace                    |

If a page is overflowing in print but looks fine on screen, the most likely causes — in order of frequency — are:

1. **CSS multi-column collapsing under fixed height.** Switch to flexbox row with pre-balanced columns (§5.5).
2. **A long list with no per-page chunking.** If the list spans multiple pages, define three explicit row constants — `ROWS_FIRST` (page with intro), `ROWS_REST` (middle pages), `ROWS_LAST_WITH_FOOT` (last page with footer) — and chunk greedily. Take everything if it fits in `ROWS_LAST_WITH_FOOT` to avoid an orphan page.
3. **Multi-line wrapping not accounted for.** Estimate row heights conservatively — assume 2 visual lines per row whenever data can wrap (long URLs, attribution lines, long item names).
4. **Cell padding too generous.** Tighten to ~0.9mm per side before reducing rows.
5. **Headers and dividers eating into the row budget.** Compute available height as `page_height − pad_top − pad_bottom − header_block − optional_footer_block` and leave a 10–15% safety margin.

When in doubt, lean on `overflow: hidden` on the page container as a defensive backstop — a misprint clips silently rather than corrupting the page count or cascading.

---

## 10. Common pitfalls — and how to avoid them

| Symptom                                                                 | Likely cause                                                                                       | Fix                                                                                                          |
|-------------------------------------------------------------------------|----------------------------------------------------------------------------------------------------|--------------------------------------------------------------------------------------------------------------|
| Blank page emitted after every page                                     | Combining modern `break-after: page` with legacy `page-break-after: always`                        | Use only the modern syntax.                                                                                  |
| A blank trailing page at the very end                                   | `.page` height equal to `@page` height; subpixel overflow                                          | Set `.page` height to **one mm less** than the page (the -1mm trick): 209mm for A4 landscape, 296mm portrait. |
| Thin white border around full-colour pages                              | -1mm gap (bottom only) OR dialog Margins not "None" (all four sides)                                | Add 5mm bleed (§6) AND document the Margins → None requirement loudly.                                       |
| Navy / amber backgrounds print white                                    | Print dialog "Background graphics" off                                                              | Tell the user to enable it. Show a banner on screen.                                                         |
| `columns: 2` collapses to one column                                    | CSS multi-column inside a fixed-height container                                                    | Compute the split at generation time. Emit a flexbox row with two `<div class="col">`.                       |
| Items disappear off the bottom of the page                              | Content overflow with `overflow: hidden` clipping silently                                          | Either expand to two pages, switch to 3-column layout, or tighten spacing. Don't remove `overflow: hidden`.  |
| Page heights differ slightly between Chrome and Safari                  | Safari respects `@page` less reliably; some default margins persist                                 | Test in Chrome. Document Chrome as the supported browser.                                                    |
| Headers and footers appear with URL/date                                | Print dialog "Headers and footers" left on                                                          | Document the requirement to disable.                                                                         |
| Text scaled or reflowed unexpectedly                                    | Print scale not at 100%                                                                             | Document the requirement to set scale to 100%.                                                                |
| `box-shadow` clipped on coloured pages                                  | `overflow: hidden` on `.page` clips pseudo-elements (but not `box-shadow` itself)                   | Use `box-shadow` spread for solid colours; for gradients, override `overflow: visible` on those pages only.  |

---

## 11. The mandatory print-dialog briefing

Every print HTML must carry an **on-screen-only** banner near the top explaining how to print correctly. This is non-optional. Without it, users print at 90% scale with default margins and a thick white frame around every page.

Minimum content of the banner:

1. Open in **Chrome**. Safari respects `@page` less reliably.
2. Press **⌘ P** (or **Ctrl P**).
3. **Destination:** Save as PDF, or the target printer.
4. **Paper size:** A4. **Layout:** Landscape (or Portrait).
5. **Margins:** **None**. Critical.
6. **Background graphics:** **Enabled**. Critical — otherwise dark backgrounds print white.
7. **Scale:** 100%.
8. **Headers and footers:** Off.

Hide the banner in print via `@media print { .print-banner { display: none !important; } }`.

---

## 12. The print stylesheet — minimum viable

The full stylesheet for a brand-aware print document runs to a few hundred lines. The irreducible core, brand-neutral, is below.

```css
@page {
  size: A4 landscape;     /* or "A4 portrait" or "A4" */
  margin: 0;
}

:root {
  --page-w: 297mm;
  --page-h: 209mm;        /* one mm less than physical */
  --pad-x: 14mm;
  --pad-t: 11mm;
  --pad-b: 10mm;
}

* { box-sizing: border-box; }

html { background: #ECECEC; }   /* off-page grey on screen */
body  { margin: 0; font-family: 'Inter', sans-serif; }

.page {
  width: var(--page-w);
  height: var(--page-h);
  overflow: hidden;
  background: #FFF;
  position: relative;
  padding: var(--pad-t) var(--pad-x) var(--pad-b);
  display: flex;
  flex-direction: column;
  page-break-inside: avoid;
  break-inside: avoid;
  break-after: page;
  margin: 0 auto 8mm;             /* on screen only — overridden in print */
  box-shadow: 0 2mm 6mm rgba(0,0,0,0.08);
}
.page:last-child { break-after: auto; }

.page-head, .page-foot { flex: 0 0 auto; }
.page-body { flex: 1 1 auto; display: flex; flex-direction: column; overflow: hidden; min-height: 0; }

.cols { display: flex; gap: 8mm; align-items: flex-start; }
.cols .col { flex: 1 1 0; min-width: 0; }
.cols .col > * { break-inside: avoid; }

.bottom-callout {
  margin-top: auto;
  padding: 3mm 5mm;
  font-style: italic;
  break-inside: avoid;
}

@media screen { body { padding: 0 0 12mm; } }

@media print {
  html, body { margin: 0 !important; padding: 0 !important; background: #FFF !important; }
  .print-banner { display: none !important; }
  .page {
    margin: 0 !important;
    padding: var(--pad-t) var(--pad-x) var(--pad-b) !important;
    box-shadow: none !important;
    break-after: page;
    break-inside: avoid;
    page-break-inside: avoid;
  }
  .page:last-child { break-after: auto; }
  a { color: inherit !important; text-decoration: none !important; }
}
```

Add the brand layer (palette, type scale, component CSS) on top of this. Add the print-bleed rules from §6 for any full-colour pages.

---

## 13. Brand-swap recipe

To take this template and apply it to a different brand:

1. **Replace the palette tokens** at the top of the `:root` block. Keep the semantic structure (brand / accent / compliance / slate / success / danger) even if the names change.
2. **Replace the type families.** Inter and Source Serif 4 are easy defaults; substitute the brand's choice via `@import` or Google Fonts.
3. **Replace the wordmark and mark.** The {{COMPANY_NAME}} wordmark is set in CSS; the mark is inline SVG. Both can be swapped in one place.
4. **Replace the cover and part-divider colours** in the print-bleed rules (§6) — the bleed must match the new background.
5. **Update the print banner** with the new brand name and document title.
6. **Audit the type scale** for the new families — different fonts have different x-heights and apparent sizes; adjust where necessary.

The geometry, slide types, and pagination rules do not change.

---

## 14. Acceptance checklist — before sign-off

Run this checklist before declaring a print HTML "done."

- [ ] **Open in Chrome.** Open the print dialog. Confirm preview renders one slide per page with no white borders (assuming Margins = None, Background graphics on).
- [ ] **No blank pages.** Scroll through the print preview end-to-end. Any blank page indicates either the -1mm trick is missing or `page-break-after: always` is being combined with `break-after: page`.
- [ ] **No clipped content.** Every page's content fits within `overflow: hidden`. Spot-check the last page of each chunked list and the longest TOC column.
- [ ] **All numbers are tabular.** Any column of numbers (stat cards, dates, page numbers, table values) uses `font-variant-numeric: tabular-nums`.
- [ ] **Action titles end with a period or have no end punctuation, consistently.** Pick one and apply throughout.
- [ ] **Bottom callout on every content page.** Sticking to the bottom of the body via `margin-top: auto`.
- [ ] **Header and footer bands present on every page** except cover and part dividers (where they're optional).
- [ ] **No legacy `page-break-after: always` anywhere.** Grep to confirm.
- [ ] **Print bleed applied** to every full-colour page.
- [ ] **Print banner present** on screen and hidden in print.
- [ ] **Box weight & elevation (§6.5).** Card-class borders run at the print-bumped values (`0.26mm` hairline / `0.78mm` top-accent / `0.91mm` standard / `1.04mm` strong) — not the screen-baseline values. Every card-class element appears in both the `box-shadow` selector list and the `print-color-adjust` selector list.
- [ ] **Shadows survive print.** Open the print dialog; preview a card-bearing page; confirm the soft drop shadow is rendered under each card. If absent, check that Background graphics is on AND `print-color-adjust: exact` is set.
- [ ] **Tag pairs balanced.** Run a quick script to confirm `<div>` / `</div>`, `<section>` / `</section>` counts match.
- [ ] **All anchor IDs resolve.** If you have an in-document TOC with `href="#..."` links, every target ID exists.

---

## 15. What this template does *not* do

For honesty.

- **No real bleed-and-crop.** No bleed marks, no trim guides. The 5mm CSS bleed is for hiding white strips, not for offset printing.
- **No CMYK.** All colours are sRGB. Convert to CMYK in the PDF prepress stage if needed.
- **No spot colour.** Same.
- **No font subsetting / embedding control.** Whatever Google Fonts gives you is what you get.
- **No automatic page-numbering against a section tree.** Page numbers and section titles in the footer are hand-typed per page. For documents over ~100 pages, write a small generator script.
- **No live cross-references.** A `Cf. §12` link is plain text. There's no auto-resolution.
- **No automatic widow / orphan control.** CSS widows/orphans support across print engines is patchy.

These limitations are usually acceptable for internal brand books, decks, reports, and proposals. They are not acceptable for trade-published books.

---

## 16. File locations and version control

Suggested structure for any project:

```
project/
  brand-guide/                        # or /report/, /deck/, /handbook/
    README.md                         # concept & rationale
    {name}.md                         # portable Markdown source
    {name}.html                       # screen edition (scrollable)
    {name}-print.html                 # print edition (A4 landscape)
    printable-slides-briefing.md      # this file (copy if reused)
```

Version both the Markdown and the HTML. The Markdown is the portable source; the HTML is the rendered presentation. Treat the print HTML and screen HTML as **co-equal renderings** of the same content, not master-and-derivative.

---

## 17. References

- The canonical print implementation: the reference brand-guide print HTML (`<brand-guide>-print.html`).
- Screen sibling for cross-checking: the reference brand-guide screen HTML (`<brand-guide>.html`).
- Portable Markdown source: the reference brand-guide Markdown (`<brand-guide>.md`).

---

*The geometry is fixed. The brand is variable. The pitfalls are predictable. Print to PDF and trust the bleed.*
