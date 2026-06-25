# HTML pattern library

Copy-and-customise HTML snippets for every content pattern the deck uses. Each section gives you:

1. The **markdown marker** that triggers the pattern (so you know when to reach for the snippet).
2. The **HTML snippet** to drop into the slide body (webview classes; the print version uses identical classes with mm-unit overrides — `references/print-spec.md` § 5 has the details).
3. A **capacity note** — how much fits comfortably.

The CSS for all of these is included in the bundled skeletons (`skeletons.md`). Don't restate it inside individual slides.

---

## §0 — Logo symbols (always include, once per file)

Drop this right after `<body>`. The wrapping `<svg>` is positioned off-viewport with `pointer-events: none` so it can never intercept touches on iPadOS. `text-anchor="end"` with x at the viewBox right edge is what makes the wordmark sit flush with the slide-head's right padding.

```html
<!-- {{COMPANY_NAME}} logo symbols — embedded for standalone rendering. Wordmark right-anchored to viewBox right edge.
     These are PLACEHOLDER wordmarks reading "COMPANY_LOGO". Replace the <text> contents (or paste the path data
     from assets/company-wordmark-*.svg) with the real wordmark. Keep the viewBox so brand.md's layout constants hold. -->
<svg xmlns="http://www.w3.org/2000/svg" style="position:fixed;top:0;left:-9999px;width:0;height:0;overflow:hidden;pointer-events:none" aria-hidden="true" focusable="false">
  <defs>
    <symbol id="company-logo-blue" viewBox="0 0 434.10657 95.619486" preserveAspectRatio="xMaxYMid meet">
      <text x="434.10657" y="62" text-anchor="end" font-family="Inter, Arial, sans-serif" font-size="46" font-weight="700" letter-spacing="-1" fill="#0B3B8C">COMPANY_LOGO</text>
    </symbol>
    <symbol id="company-logo-white" viewBox="0 0 434.10657 95.619486" preserveAspectRatio="xMaxYMid meet">
      <text x="434.10657" y="62" text-anchor="end" font-family="Inter, Arial, sans-serif" font-size="46" font-weight="700" letter-spacing="-1" fill="#F9F9F9">COMPANY_LOGO</text>
    </symbol>
  </defs>
</svg>
```

Then every slide-head references the logo with:

```html
<span class="brand" aria-label="{{COMPANY_NAME}}">
  <svg class="brand-svg blue" viewBox="0 0 434.10657 95.619486" role="img" aria-hidden="true"><use href="#company-logo-blue"/></svg>
  <svg class="brand-svg white" viewBox="0 0 434.10657 95.619486" role="img" aria-hidden="true"><use href="#company-logo-white"/></svg>
</span>
```

CSS picks which is visible based on slide background — see `skeletons.md` § Webview CSS.

---

## §1 — Cover

Markdown: the cover is the document's title block plus a `Meta strip` line.

```html
<section class="slide cover active" data-title="Cover" data-kind="cover">
  <header class="slide-head">
    <span>{{HEADER_LABEL}}</span>           <!-- e.g. "{{COMPANY_NAME}} · {{RECIPIENT}} Conversation" -->
    <span>{{HEADER_RIGHT}}</span>            <!-- e.g. "2026-05-28 · 30 minutes" -->
  </header>
  <div class="slide-body">
    <div class="cover-center">
      <span class="tag">{{POSITIONING_CHIP}}</span>   <!-- e.g. "{{COMPANY_TAGLINE}}" -->
      <h1>{{COVER_TITLE}}.</h1>                       <!-- e.g. "{{COMPANY_NAME}}." -->
      <p class="lede">{{COVER_LEDE}}</p>
    </div>
    <div class="cover-meta-row">
      <div>Document<span>{{DOC_NAME}}</span></div>
      <div>Edition<span>{{EDITION}}</span></div>
      <div>Presenter<span>{{PRESENTER_NAME}}<br>{{PRESENTER_EMAIL}}</span></div>
      <div>{{COL4_LABEL}}<span>{{COL4_VALUE}}</span></div>   <!-- Location / Format -->
    </div>
  </div>
  <footer class="slide-foot">
    <span>→ to begin · ? for keyboard help</span>
    <span>Cover</span>
  </footer>
</section>
```

Capacity: cover title ≤ 2 words, lede ≤ 4 lines.

---

## §2 — Part divider

```html
<section class="slide part-divider" data-title="Part {{ROMAN}} · {{PART_NAME}}" data-kind="divider">
  <header class="slide-head"><span>Part {{ROMAN}}</span><span class="brand">…logo SVGs…</span></header>
  <div class="slide-body">
    <div class="part-number">Part {{ROMAN}}</div>
    <h1>{{PART_TITLE}}.</h1>
    <p class="part-lede">{{PART_LEDE}}</p>
    <div class="part-chips">
      <span class="chip">Slides {{N}}–{{M}}</span>
      <span class="chip">Minutes {{A}}–{{B}}</span>
      <span class="chip">{{TOPIC_CHIPS}}</span>
    </div>
  </div>
  <footer class="slide-foot"><span>Part {{ROMAN}} — {{PART_NAME}}</span><span>Divider</span></footer>
</section>
```

---

## §3 — Content slide skeleton

Every content slide starts here. The body region in the middle is where one of §4–§14 plugs in.

```html
<section class="slide" data-title="{{NN}} · {{SLIDE_LABEL}}" data-kind="content">
  <header class="slide-head"><span>{{NN}} · {{SLIDE_LABEL}}</span><span class="brand">…logo SVGs…</span></header>
  <div class="slide-body">
    <h1>{{ACTION_TITLE}}</h1>
    <div class="accent-bar"></div>
    <p class="subtitle">{{SUBTITLE}}</p>

    {{BODY_PATTERN}}

    <div class="bottom-callout {{VARIANT}}"><em>{{CALLOUT_TEXT}}</em></div>
  </div>
  <footer class="slide-foot">
    <span>{{NN}} · {{SLIDE_LABEL}}</span>
    {{#CONFIDENTIAL}}<span class="confidential">CONFIDENTIAL — {{NAME}} {{VERSION}}</span>{{/CONFIDENTIAL}}
    <span>Slide {{N}} of {{TOTAL}}</span>
  </footer>
</section>
```

`{{VARIANT}}` = `navy` (default — omit class) | `teal` | `amber`. Reserve amber for compliance / deadlines.

The middle `confidential` span is **optional** — include it when the deck is targeted at a single named recipient (see `SKILL.md` § Step 1b). When omitted, the surrounding `.slide-foot` CSS reverts to the two-column flex layout; when included, the `skeletons.md` § Confidentiality footer CSS turns the footer into a three-column grid so the strap sits centred between the existing labels.

---

## §4 — Three-stat

Markdown marker: `**Three-stat.**` followed by three bullets, each beginning with a big number.

```html
<div class="three-stat">
  <div class="stat-card">
    <div class="big">{{BIG_1}}</div>
    <div class="label">{{LABEL_1}}</div>
    <div class="source">{{SOURCE_1}}</div>
  </div>
  <!-- × 3 -->
</div>
```

Capacity: 3 cards. Each label ≤ 18 words, each source ≤ 10 words.

---

## §5 — Three-windows

Markdown marker: `**Three-windows pattern.**` then three blocks each prefixed by a colour role: `*Navy — …*`, `*Amber — …*`, `*Teal — …*`.

```html
<div class="three-windows">
  <div class="window-card navy">
    <div class="head">{{HEAD_TEXT_1}}</div>
    <p>{{BODY_1}}</p>
    <div class="date">{{TAGLINE_1}}</div>
  </div>
  <div class="window-card amber">…</div>
  <div class="window-card teal">…</div>
</div>
```

Capacity: 3 cards, body ≤ 4 short paragraphs per card.

---

## §6 — Compare-and-contrast

Markdown marker: `**Compare-and-contrast.**` with `*Left — Without …*` and `*Right — With …*` (or any two columns).

```html
<div class="compare">
  <div class="compare-card without">
    <h4>{{LEFT_HEAD}}</h4>
    <ul><li>…</li></ul>
  </div>
  <div class="compare-card with">
    <h4>{{RIGHT_HEAD}}</h4>
    <ul><li>…</li></ul>
  </div>
</div>
```

If the slide has a centre cycle-time callout (e.g. "2–3 days → 30 minutes"), append:

```html
<div class="compare-callout">{{TEXT}}</div>
```

Capacity: 2 cards, 4–6 bullets per card.

---

## §7 — Two columns (generic)

Markdown marker: `**Two columns.**` with `*Left — …*` and `*Right — …*`.

```html
<div class="cols">
  <div class="col">
    <h2>{{LEFT_HEAD}}</h2>
    <ul>…</ul>
  </div>
  <div class="col">
    <h2>{{RIGHT_HEAD}}</h2>
    <ul>…</ul>
  </div>
</div>
```

---

## §8 — Four-pillars

Markdown marker: `**Four-pillar pattern.**` plus a table of `| # | Pillar | What it does |`.

```html
<div class="four-pillars">
  <div class="pillar">
    <div class="head"><span class="num">01</span><span class="name">{{NAME}}</span></div>
    <div class="body"><p>{{DESC}}</p></div>
  </div>
  <!-- × 4 -->
</div>
```

Capacity: 4 cards, 3–5 bullets per card.

---

## §9 — Architecture pipeline

Markdown marker: `**Pipeline — seven stages with arrows.**` with an ordered list of stages.

```html
<div class="arch-pipeline">
  <div class="arch-stage">
    <svg class="icon">…</svg>
    <div class="num">01</div>
    <div class="name">{{STAGE_NAME}}</div>
  </div>
  <!-- × 7; CSS adds the → between each pair -->
</div>
```

Below the pipeline, optionally a `.arch-blocks` row with grouped feature cards (Frontend / API / Data).

Capacity: 7 stages plus an optional 3-group building-block row.

---

## §10 — Lane stack (3 horizontal layers)

Used to show where the product sits in a layered architecture. Markdown marker: `**Three layers — top to bottom**` plus three named layers.

```html
<div class="lane-stack">
  <div class="lane-row lane-top">
    <div class="lane-title">
      <svg class="lane-icon">…</svg>
      <div class="lane-name">{{TOP_LAYER_NAME}}</div>
    </div>
    <div class="lane-content"><p><strong>{{BOLD_LEAD}}.</strong> {{REST}}</p></div>
  </div>
  <div class="lane-row lane-mid">
    <div class="lane-title">…<div class="lane-name">{{MID_LAYER_NAME}}<span class="lane-sub">{{COMPANY_NAME}}</span></div></div>
    <div class="lane-content">…</div>
  </div>
  <div class="lane-row lane-bottom">
    <div class="lane-title">…<div class="lane-name">{{BOTTOM_LAYER_NAME}}</div></div>
    <div class="lane-content">…</div>
  </div>
</div>
```

The middle (teal, `.lane-mid`) is visually heaviest by design — put the layer the deck is about (typically the product, `{{COMPANY_NAME}}`) in the middle.

---

## §11 — Big-date callout (+ adjacent panel)

Markdown marker: `**Date callout (with strikethrough).**` plus a `**Why…**` panel.

```html
<div class="big-date">
  <div class="date-card">
    <div class="label">{{ACT_NAME}}</div>
    <div class="date"><span class="date-strike">{{OLD_DATE}}</span>{{NEW_DATE}}</div>
    <div class="sub">{{PROVISO}}</div>
  </div>
  <div class="deferral-card">
    <h4>{{PANEL_HEAD}}</h4>
    <p><b>{{LEAD_BOLD}}.</b> {{BODY}}</p>
  </div>
</div>
```

Important: `.deferral-card p { max-width: none }` — otherwise the global `.page p { max-width: 145mm }` rule eats the right side of the print version and pushes the bottom callout off-page (real bug from slide 15).

---

## §12 — Methodology table

Markdown marker: a long table starting with `| Methodology | What it delivers | Where in {{COMPANY_NAME}} it lives | When it runs |`.

```html
<table class="method-table">
  <thead>
    <tr><th>Methodology</th><th>What it delivers</th><th>Where it lives</th><th>When it runs</th></tr>
  </thead>
  <tbody>
    <tr>
      <td><b>{{NAME}}</b><span class="src">{{SOURCE}}</span></td>
      <td data-label="What it delivers">{{DELIVERS}}</td>
      <td data-label="Where it lives">{{LIVES}}</td>
      <td data-label="When it runs">{{WHEN}}</td>
    </tr>
    <!-- × N rows -->
  </tbody>
</table>
```

Print version: scope the body-cell font down. `.page .method-table td { font-size: 7.5pt }` with the first column kept at 9pt. The `.page` selector beats the generic `.page table td` rule — required for the override to apply.

`data-label` attributes are required on every `<td>` after the first column so the table can collapse to cards on phone (≤ 640 px). Without them, mobile users see unlabelled values.

---

## §13 — QA-list (objections + answers)

Markdown marker: a sequence of `*Objection — "…"*` followed by `**Answer.**`.

```html
<div class="qa-list">
  <div class="qa">
    <div class="obj">"{{OBJECTION}}"</div>
    <div class="ans">{{ANSWER}}</div>
  </div>
  <!-- × N -->
</div>
```

Capacity: 5–6 objections at 10–11 px body fits the 720 px frame; for 7+ drop the font half a point or split across two slides.

---

## §14 — Lens extras prose row (ROI slide pattern)

A continuous-prose block listing additional named items with bold labels and italic supporting text, separated by middle-dots. Used on the ROI slide ("Where is the money").

```html
<div class="lens-extras">
  <span class="label">{{EYEBROW}}</span>
  <b>{{LENS_1_NAME}}</b> · <em>{{LENS_1_MATH}}</em>.
  <b>{{LENS_2_NAME}}</b> · <em>{{LENS_2_MATH}}</em>.
  <!-- … -->
</div>
```

Use when you want to name 4–6 additional items without giving each one a full card. Markdown marker: a paragraph beginning `**Other lenses CFOs ask about** *(named, not elaborated …)*`.

---

## §15 — Bottom callout

Required on every content slide. Already in §3's skeleton. Variants:

```html
<div class="bottom-callout"><em>…</em></div>            <!-- navy default -->
<div class="bottom-callout teal"><em>…</em></div>
<div class="bottom-callout amber"><em>…</em></div>
```

Italic one-liner. Compresses the slide's claim. Never repeats the title.

---

## §16 — Sources / appendix slide

Markdown marker: `# Slide M · Sources`.

```html
<div class="cols">
  <div class="col">
    <h3>{{GROUP_1}}</h3>
    <ul><li>…</li></ul>
    <h3>{{GROUP_2}}</h3>
    <ul><li>…</li></ul>
  </div>
  <div class="col">
    <h3>{{GROUP_3}}</h3>
    <ul><li>…</li></ul>
  </div>
</div>
```

Same `.cols` pattern as §7, used to lay out a long bibliography across two columns.

## §17 — Confidentiality footer (every slide, when enabled)

For single-recipient pitches the footer carries a centred strap with the template `CONFIDENTIAL — [NAME] [VERSION]`. Apply to every slide (cover, dividers, content) by inserting a third span between the existing two:

```html
<footer class="slide-foot">
  <span>{{LEFT_LABEL}}</span>
  <span class="confidential">CONFIDENTIAL — {{NAME}} {{VERSION}}</span>
  <span>{{RIGHT_LABEL}}</span>
</footer>
```

Same shape on the print companion (`<footer class="page-foot">`). The CSS in `skeletons.md` § Confidentiality footer CSS handles the three-column grid and switches strap colour (red on light, amber on dark) so the strap reads as a watermark rather than competing with the body text.

**Example values.**

| Markdown source title | `{{NAME}}` | `{{VERSION}}` | Rendered strap |
|---|---|---|---|
| `{{COMPANY_NAME}} — Client Pitch` (Edition 1.3) | `CLIENT NAME` | `VERSION 1.3` | `CONFIDENTIAL — CLIENT NAME VERSION 1.3` |
| `Board Proposal` (Edition 2.0) | `BOARD OF DIRECTORS` | `VERSION 2.0` | `CONFIDENTIAL — BOARD OF DIRECTORS VERSION 2.0` |
| Internal-only quarterly review (Edition 1.0) | `INTERNAL ONLY` | `VERSION 1.0` | `CONFIDENTIAL — INTERNAL ONLY VERSION 1.0` |

If the deck has no specific audience (public marketing collateral, conference talk), **omit the strap entirely** — the footer reverts to its original two-column layout.
