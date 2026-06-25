# {{COMPANY_NAME}} Web Viewer — format specification

**Version:** 1.3
**Date:** 2026-05-27
**Status:** Reference specification. Reusable as prompt or context.
**Companion files:** `{{COMPANY_NAME}}-brand-guide.md`, `{{COMPANY_NAME}}-brand-guide.html` (visual reference, if maintained).

**What changed in 1.3.** iPad robustness pass. The 1.1 implementation worked on iPhone (phone stack mode) but had several iPad failure modes when opened from Messages attachments or the Files app — touch swipes occasionally dropped, hardware-keyboard events sometimes didn't reach `document`, and inline `<symbol>` blocks added by later pitches could intercept the focus chain. 1.3 hardens all three. See §14.7.

**What changed in 1.2.** Boxes carry more weight. Every visible card-class border is ~30% thicker than the 1.1 baseline, and a small two-stop drop shadow lifts the card off the slate-50 surface so the edges read at presenter-screen distance. See §7.9.

**What changed in 1.1.** Added cross-device responsiveness: touch gestures + a focus-trap input that captures hardware keyboard events on iPadOS Safari; responsive frame scaling for tablets; phone "stack mode" — horizontal scroll-snap between slides, vertical scroll within each slide's content — for iPhone-class viewports. See §14.

This document specifies the {{COMPANY_NAME}} **Web Viewer** — a browser-renderable slide-deck format used for decks, memos, briefs, and any long-form structured argument that should be navigable as slides. It is the on-screen sibling of the **Print HTML** (A4 landscape) format; the two are always produced as a pair.

When asked to produce a deck, pitch, strategy document, brief, market memo, or any long-form structured argument that benefits from slide-by-slide navigation, build to this spec.

---

## 1. Purpose and posture

The Web Viewer presents content as a sequence of fixed-aspect slides (1280×720, 16:9) rendered in the browser. Each slide is scaled to fit (or fill) the viewport, navigated by keyboard or mouse, and addressable by URL hash for deep-linking. The format is purpose-built for the brand's preferred argument shape: short, dense, action-titled units that read independently and accumulate into a thesis.

Reach for the Web Viewer format when:

- The artefact is **structured** (parts, sections, slides) rather than continuous prose.
- The reader benefits from **slide-by-slide control** rather than scroll-through.
- The same content should later **print as A4 landscape** (a deck people walk out of a room with).
- The brand's **content patterns** (three-stat, three-windows, four-pillars, compare-and-contrast, big-date callout, layer-stack, arch-pipeline, qa-list, bottom-callout) are the natural carriers of the argument.

Do not use the Web Viewer format for short emails, essays, single-page reports, README files, or anything that reads better as continuous prose. Use markdown or a Word/PDF format instead.

---

## 2. The file pair

Every Web Viewer deliverable is produced as a pair:

| File | Suffix | Purpose |
|------|--------|---------|
| **Web Viewer** | `.html` | On-screen navigation. Frame-scaled. Keyboard nav. Fullscreen modes. |
| **Print HTML** | `-print.html` | A4 landscape paginated. One slide per A4 page. Print-ready. |

Both files share the same content, brand discipline, and visual system. The Web Viewer links to the print file via the `P` key and a print HUD chip. The print file does **not** link back — printed sheets carry no reference to the screen view.

Optional companion:

| File | Suffix | Purpose |
|------|--------|---------|
| **Markdown source** | `.md` | Authoring source. Carries the full content in flat form. |

When all three exist, the markdown source is canonical; the two HTML files are renderings of it. When only the HTMLs exist, they are themselves canonical and edits happen there.

Naming convention (matches the existing internal pattern):

```
<Topic>-<Version>.md
<Topic>-<Version>.html
<Topic>-<Version>-print.html
```

Examples:

```
strategy-2.1.md
strategy-2.1.html
strategy-2.1-print.html

market-development.md
market-development.html
market-development-print.html
```

---

## 3. Visual system

Inherits the brand guide defined in `brand.md` §Palette/§Typography. Summarised here for self-containment:

### Palette (CSS custom properties)

```css
--navy-900:#06245A; --navy-700:#0B3B8C; --navy-500:#2E5BBF; --navy-50:#EAF1FB;
--teal-700:#16998A; --teal-500:#1FB8A0; --teal-50:#EAF7F4;
--amber-700:#C7770A; --amber-500:#F39C12; --amber-50:#FCEDB7;
--slate-950:#0F1422; --slate-800:#1F2A3D; --slate-600:#4A5A70;
--slate-400:#8A95A8; --slate-200:#E2E6EC; --slate-100:#EEF1F5; --slate-50:#F8FAFC;
--white:#FFFFFF; --success:#28A745; --danger:#D9534F;
```

**Lead with Navy. Accent with Teal. Reserve Amber for compliance and deadlines.** No more than three colour roles on a single slide.

### Typography

```css
--sans: 'Inter', -apple-system, BlinkMacSystemFont, 'Segoe UI', sans-serif;
--serif: 'Source Serif 4', Georgia, 'Times New Roman', serif;
```

Inter for everything except long-form editorial passages (founder letters, manifestos, printed monographs). Source Serif 4 only where it earns its keep. Tabular figures (`font-feature-settings: "tnum" 1`) on stat cards, tables, dates, and any number-heavy region.

### Type scale (slide-adjusted)

The Web Viewer uses a tightened scale relative to the brand guide's web defaults — slides are dense and the 1280×720 frame benefits from smaller type than a long-form page.

| Element | Size | Weight | Line height |
|---------|------|--------|-------------|
| Cover H1 | 104px | 700 | 1.02 |
| Part divider H1 | 80px | 700 | 1.04 |
| Slide H1 (action title) | 28px | 700 | 1.14 |
| Subtitle | 13.5px | 400 | 1.42 |
| H2 (section head) | 14.5px | 600 | 1.25 |
| H3 (label) | 12.5px | 600 | — |
| Body | 11.5px | 400 | 1.5 |
| Body small | 10.5–11px | 400 | 1.35–1.4 |
| Stat big number | 42px | 700 | 1.0 |
| Caption / metadata | 9.5–10px | 500 | 1.4 |

These sizes are calibrated for slide density. Cover and part dividers run larger; in-content cards (stat, window, pillar) run smaller. Tighten further only if a slide is overflowing.

---

## 4. Slide architecture

### 4.1 The frame

Every slide renders into a fixed 1280×720 frame (`--slide-w`, `--slide-h`). The frame is centred in the viewport and scaled via CSS `transform`. JavaScript recomputes the scale on `resize` and on mode change.

```html
<div class="stage" id="stage">
  <div class="frame" id="frame">
    <section class="slide active" data-title="...">...</section>
    <section class="slide" data-title="...">...</section>
    <!-- … -->
  </div>
</div>
```

Only one `<section class="slide">` carries the `.active` class at a time. The rest are `display: none`.

### 4.2 Three scaling modes

| Mode | Trigger | Behaviour |
|------|---------|-----------|
| **Window** (default) | On load | Uniform scale to fit the frame inside the viewport with a comfortable margin (`Math.min(vw/1280, vh/720, 1.4)`). Border-radius and drop shadow visible. |
| **Fit-screen fullscreen** | `f` | Browser fullscreen + uniform scale to fit (letterboxed). Page background tinted dark; frame retains border-radius. |
| **Fill-screen fullscreen** | `Shift+F` | Browser fullscreen + **non-uniform** scale (`scale(vw/1280, vh/720)`). No letterbox; frame fills viewport. Border-radius and box-shadow removed. |

In fill-screen mode the aspect ratio can stretch slightly to fill non-16:9 displays. This is the desired behaviour — the brand prefers a covered screen over a letterboxed one in that mode. Use `f` when aspect fidelity matters (filming, screenshot); use `Shift+F` when the room matters more than the pixels.

### 4.3 Slide structure

Each slide is a `<section class="slide">` with three regions:

```html
<section class="slide active" data-title="Slide 03 · The category gap" data-kind="content">
  <header class="slide-head">
    <span>Part I · The opportunity</span>
    <span class="brand">{{COMPANY_NAME}}</span>
  </header>
  <div class="slide-body">
    <!-- action title, content, bottom callout -->
  </div>
  <footer class="slide-foot">
    <span>03 · The category gap</span>
    <span>Slide 3 of 56</span>
  </footer>
</section>
```

- `data-title` is required and powers the overview grid and HUD title.
- `data-kind` is one of `cover` | `divider` | `content`. Used for overview rendering.
- The brand mark in the head (`.brand` with `::before` pseudo) is a small ring + chord glyph — the brand-guide's documented mark.
- The footer shows the action title (left) and "Slide X of Y" (right). Part dividers and cover override these.

### 4.4 Slide kinds

| Kind | Class | Use |
|------|-------|-----|
| Cover | `.slide.cover` | First slide. Navy-900 background, large white H1, teal pill tag, four-column meta row. |
| Part divider | `.slide.part-divider` | One per part. Navy gradient background, large H1, lede, chip row showing slide range. |
| Content | `.slide` | All argument-carrying slides. Slate-50 background, action title in Navy 700, content patterns below. |

Always begin with a cover. Always group content slides under part dividers if the deck has more than ~12 slides. Number content slides continuously (1, 2, 3 …); part dividers and cover are not numbered.

---

## 5. Navigation

### 5.1 Keyboard map

| Key | Action |
|-----|--------|
| `→`, `Space`, `PageDown` | Next slide |
| `←`, `Backspace`, `PageUp` | Previous slide |
| `Home` | First slide |
| `End` | Last slide |
| `O` | Toggle overview (thumbnail grid of all slides) |
| `Esc` | Close overview (or exit fullscreen) |
| `f` | Fullscreen — fit (uniform, letterboxed) |
| `Shift+F` | Fullscreen — fill (non-uniform, no letterbox) |
| `P` | Open the print edition in a new tab |
| `?` | Show keyboard help |

### 5.2 Mouse

- Click in the **left quarter** of the stage (outside the frame) → previous slide.
- Click in the **right quarter** of the stage (outside the frame) → next slide.
- Click inside the frame or HUD → no-op (allows text selection and copy).
- Click an overview thumbnail → jump to that slide and close overview.

### 5.3 Deep linking

The URL hash is updated on every navigation: `#slide-N` where N is the zero-based index. On load, the hash is honoured. Authors can link from external documents to specific slides:

```
https://example.com/strategy-2.1.html#slide-14
```

### 5.4 HUD elements

| Element | Position | Content |
|---------|----------|---------|
| Progress bar | Top edge, 2px | Teal 500 fill, width = current/total |
| Slide title + position | Bottom-left | `<current title> · <NN of NN>` |
| Key hints | Bottom-right | `← →` `O` `f / ⇧F` `P` `?` |

In fill-screen mode the HUD recolours to slate-on-light to remain legible against the lighter slate-50 frame.

---

## 6. HTML skeleton

The minimal viable Web Viewer. Drop content slides into the marked region; everything else is invariant.

```html
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>{{Document name}} · Web Viewer</title>
<meta name="description" content="{{One-line description}}">
<link rel="preconnect" href="https://fonts.googleapis.com">
<link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
<link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;500;600;700&family=Source+Serif+4:ital,wght@0,400;0,600;1,400&display=swap" rel="stylesheet">
<style>
  /* See §7 — full CSS goes here */
</style>
</head>
<body>

<div class="progress"><div class="bar" id="progress-bar" style="width:0"></div></div>

<div class="stage" id="stage">
<div class="frame" id="frame">

  <!-- COVER -->
  <section class="slide cover active" data-title="Cover" data-kind="cover">
    <!-- cover content -->
  </section>

  <!-- PART I DIVIDER -->
  <section class="slide part-divider" data-title="Part I · …" data-kind="divider">
    <!-- divider content -->
  </section>

  <!-- CONTENT SLIDES -->
  <section class="slide" data-title="01 · …" data-kind="content">
    <!-- slide content -->
  </section>

  <!-- … repeat for every content slide and part divider … -->

</div>
</div>

<div class="hud">
  <div class="hud-left"><span id="title-display"></span><span class="pos" id="pos-display"></span></div>
  <div class="hud-right">
    <span class="hint"><span class="key">←</span><span class="key">→</span></span>
    <span class="hint"><span class="key">O</span> overview</span>
    <span class="hint"><span class="key">f</span> fit · <span class="key">⇧F</span> fill</span>
    <span class="hint"><span class="key">P</span> print</span>
    <span class="hint"><span class="key">?</span></span>
  </div>
</div>

<div class="overview" id="overview">
  <div class="close-hint">Esc to close</div>
  <h2>{{Document name}}</h2>
  <p class="sub">{{NN slides · NN parts}}</p>
  <div class="overview-grid" id="overview-grid"></div>
</div>

<script>
  /* See §8 — full JS goes here */
</script>
</body>
</html>
```

---

## 7. CSS essentials

The full stylesheet for the Web Viewer. Drop into the `<style>` block of the skeleton above.

### 7.1 Page and frame

```css
:root {
  --navy-900:#06245A; --navy-700:#0B3B8C; --navy-500:#2E5BBF; --navy-50:#EAF1FB;
  --teal-700:#16998A; --teal-500:#1FB8A0; --teal-50:#EAF7F4;
  --amber-700:#C7770A; --amber-500:#F39C12; --amber-50:#FCEDB7;
  --slate-950:#0F1422; --slate-800:#1F2A3D; --slate-600:#4A5A70;
  --slate-400:#8A95A8; --slate-200:#E2E6EC; --slate-100:#EEF1F5; --slate-50:#F8FAFC;
  --white:#FFFFFF; --success:#28A745; --danger:#D9534F;
  --sans:'Inter',-apple-system,BlinkMacSystemFont,'Segoe UI',sans-serif;
  --serif:'Source Serif 4',Georgia,serif;
  --slide-w:1280px; --slide-h:720px;
}
* { box-sizing: border-box; }
html, body {
  margin: 0; padding: 0; height: 100%;
  font-family: var(--sans);
  background: var(--slate-950);
  color: var(--slate-600);
  font-feature-settings: "ss01" 1;
  -webkit-font-smoothing: antialiased;
  text-rendering: optimizeLegibility;
  overflow: hidden;
}
.stage {
  position: fixed; inset: 0;
  display: flex; align-items: center; justify-content: center;
  background: #15181F;
}
.stage::before, .stage::after {
  content: ''; position: absolute; top: 0; bottom: 0; width: 25%;
  z-index: 5; cursor: pointer;
}
.stage::before { left: 0; }
.stage::after  { right: 0; }
.frame {
  width: var(--slide-w); height: var(--slide-h);
  background: var(--slate-50);
  position: relative;
  box-shadow: 0 30px 80px rgba(0,0,0,0.4);
  transform-origin: center center;
  border-radius: 4px;
  overflow: hidden;
}
```

### 7.2 Slide layout

```css
.slide {
  position: absolute; inset: 0;
  display: none; flex-direction: column;
  padding: 36px 64px 32px;
  background: var(--slate-50);
  animation: fade-in 280ms ease;
}
.slide.active { display: flex; }
@keyframes fade-in {
  from { opacity: 0; transform: translateY(6px); }
  to   { opacity: 1; transform: translateY(0); }
}

.slide-head {
  display: flex; justify-content: space-between; align-items: center;
  font-size: 10.5px; letter-spacing: 0.1em; text-transform: uppercase;
  color: var(--slate-400); margin-bottom: 14px;
  font-variant-numeric: tabular-nums; font-weight: 500;
  border-bottom: 1px solid var(--slate-200);
  padding-bottom: 10px; flex: 0 0 auto;
}
.slide-head .brand {
  display: inline-flex; align-items: center; gap: 8px;
  color: var(--slate-800); letter-spacing: -0.005em;
  text-transform: none; font-weight: 600; font-size: 13px;
}
.slide-head .brand::before {
  content: ''; width: 14px; height: 14px; border-radius: 50%;
  border: 1.5px solid var(--navy-700); display: inline-block; position: relative;
  background: linear-gradient(to bottom right,
    transparent 49%, var(--teal-500) 49%, var(--teal-500) 51%, transparent 51%);
}
.slide-body { flex: 1 1 auto; display: flex; flex-direction: column; min-height: 0; }
.slide-foot {
  display: flex; justify-content: space-between; align-items: center;
  font-size: 9.5px; letter-spacing: 0.08em; text-transform: uppercase;
  color: var(--slate-400); font-variant-numeric: tabular-nums;
  border-top: 1px solid var(--slate-200);
  padding-top: 10px; margin-top: 12px; flex: 0 0 auto;
}
```

### 7.3 Typography

```css
.slide h1 {
  font-size: 28px; font-weight: 700; line-height: 1.14; letter-spacing: -0.018em;
  color: var(--navy-700); margin: 0 0 8px;
}
.slide .subtitle {
  font-size: 13.5px; line-height: 1.42; color: var(--slate-600);
  margin: 0 0 14px; max-width: 1000px; font-weight: 400;
}
.slide .accent-bar {
  width: 48px; height: 2.5px; background: var(--teal-500); margin: -2px 0 10px;
}
.slide h2 {
  font-size: 14.5px; font-weight: 600; color: var(--slate-950);
  margin: 8px 0 5px; letter-spacing: -0.005em; line-height: 1.25;
}
.slide h3 {
  font-size: 12.5px; font-weight: 600; color: var(--slate-950); margin: 8px 0 3px;
}
.slide p {
  margin: 0 0 7px; font-size: 11.5px; line-height: 1.5;
}
.slide p strong { color: var(--slate-800); font-weight: 600; }
.slide em { font-style: italic; }
.slide ul, .slide ol { padding: 0; margin: 0 0 8px; list-style: none; }
.slide ul li {
  position: relative; padding-left: 14px; margin: 3px 0;
  font-size: 11px; line-height: 1.38;
}
.slide ul li::before {
  content: ''; position: absolute; left: 0; top: 6px;
  width: 5px; height: 5px; border-radius: 50%; background: var(--navy-700);
}
.slide ol { counter-reset: cog-counter; }
.slide ol li {
  counter-increment: cog-counter;
  position: relative; padding-left: 24px; margin: 7px 0;
  font-size: 12px; line-height: 1.48;
}
.slide ol li::before {
  content: counter(cog-counter, decimal-leading-zero);
  position: absolute; left: 0; top: 0;
  color: var(--teal-500); font-weight: 600;
  font-variant-numeric: tabular-nums; font-size: 11px;
}
.slide table { width: 100%; border-collapse: collapse; margin: 4px 0 8px; font-size: 11px; }
.slide table th {
  text-align: left; font-weight: 600; color: var(--slate-800);
  padding: 4px 12px 4px 0; border-bottom: 1px solid var(--slate-200);
  font-size: 10px; letter-spacing: 0.08em; text-transform: uppercase;
}
.slide table td {
  padding: 4px 12px 4px 0; border-bottom: 1px solid var(--slate-100);
  vertical-align: top; color: var(--slate-600);
  font-variant-numeric: tabular-nums; font-size: 11px; line-height: 1.36;
}
.slide table td:first-child { color: var(--slate-800); font-weight: 500; }
.slide blockquote {
  margin: 6px 0 10px; padding: 10px 18px;
  background: var(--navy-50); border-left: 3px solid var(--navy-700);
  font-style: italic; color: var(--slate-800);
  font-size: 11.5px; line-height: 1.45; max-width: 920px;
}
.slide code {
  font-family: ui-monospace, 'SF Mono', Menlo, monospace;
  font-size: 10.5px; background: var(--slate-100);
  padding: 1px 5px; border-radius: 3px; color: var(--navy-700);
}
.bottom-callout {
  margin-top: auto; padding: 9px 16px;
  background: var(--navy-50); border-left: 4px solid var(--navy-700);
  font-style: italic; color: var(--slate-800);
  font-size: 12.5px; line-height: 1.42;
}
.bottom-callout.teal  { background: var(--teal-50);  border-left-color: var(--teal-500); }
.bottom-callout.amber { background: var(--amber-50); border-left-color: var(--amber-500); }
.cols { display: flex; gap: 22px; align-items: flex-start; }
.cols .col { flex: 1 1 0; min-width: 0; }
```

### 7.4 Cover and part dividers

```css
.slide.cover {
  background: var(--navy-900); color: var(--white); padding: 60px 80px;
}
.slide.cover .slide-head { color: rgba(255,255,255,0.55); border-bottom-color: rgba(255,255,255,0.15); }
.slide.cover .slide-head .brand { color: var(--white); }
.slide.cover .slide-head .brand::before { border-color: var(--white); }
.slide.cover .slide-foot { color: rgba(255,255,255,0.4); border-top-color: rgba(255,255,255,0.15); }
.slide.cover .cover-center { margin-top: auto; margin-bottom: auto; }
.slide.cover .tag {
  display: inline-block; font-size: 11px; letter-spacing: 0.16em; text-transform: uppercase;
  color: var(--teal-500); border: 1px solid var(--teal-500);
  padding: 7px 16px; border-radius: 999px; margin-bottom: 24px; font-weight: 500;
}
.slide.cover h1 {
  font-size: 104px; font-weight: 700; letter-spacing: -0.028em; line-height: 1.02;
  margin: 0 0 20px; color: var(--white);
}
.slide.cover .lede {
  font-size: 18px; color: rgba(255,255,255,0.78);
  max-width: 880px; line-height: 1.5; font-weight: 400;
}
.slide.cover .cover-meta-row {
  display: grid; grid-template-columns: repeat(4, 1fr); gap: 24px;
  font-size: 9.5px; letter-spacing: 0.1em; text-transform: uppercase;
  color: rgba(255,255,255,0.45); margin-top: 40px; padding-top: 18px;
  border-top: 1px solid rgba(255,255,255,0.15);
}
.slide.cover .cover-meta-row div span {
  display: block; color: var(--white); font-size: 13px;
  letter-spacing: 0; text-transform: none; margin-top: 5px;
  font-weight: 500; font-variant-numeric: tabular-nums; line-height: 1.35;
}

.slide.part-divider {
  background: linear-gradient(135deg, var(--navy-900) 0%, var(--navy-700) 100%);
  color: var(--white); padding: 60px 80px;
}
.slide.part-divider .slide-head { color: rgba(255,255,255,0.6); border-bottom-color: rgba(255,255,255,0.15); }
.slide.part-divider .slide-head .brand { color: var(--white); }
.slide.part-divider .slide-head .brand::before { border-color: var(--white); }
.slide.part-divider .slide-foot { color: rgba(255,255,255,0.4); border-top-color: rgba(255,255,255,0.15); }
.slide.part-divider .slide-body { justify-content: center; }
.slide.part-divider .part-number {
  font-size: 12px; letter-spacing: 0.22em; text-transform: uppercase;
  color: var(--teal-500); margin-bottom: 14px; font-weight: 600;
}
.slide.part-divider h1 {
  font-size: 80px; font-weight: 700; letter-spacing: -0.028em;
  line-height: 1.04; color: var(--white); margin: 0 0 22px;
}
.slide.part-divider .part-lede {
  font-size: 16px; line-height: 1.5; color: rgba(255,255,255,0.72);
  max-width: 880px; font-weight: 400; margin-bottom: 24px;
}
.slide.part-divider .part-chips {
  display: flex; gap: 10px; flex-wrap: wrap;
}
.slide.part-divider .part-chips .chip {
  font-size: 10px; letter-spacing: 0.1em; text-transform: uppercase;
  color: rgba(255,255,255,0.85); border: 1px solid rgba(255,255,255,0.25);
  padding: 5px 12px; border-radius: 999px; font-variant-numeric: tabular-nums;
}
```

### 7.5 Content patterns

These are the same patterns named in the brand guide §4. The CSS below is the slide-tuned version of each.

```css
/* Three-stat */
.three-stat { display: grid; grid-template-columns: repeat(3, 1fr); gap: 14px; margin: 8px 0; }
.stat-card {
  background: var(--white); border: 1px solid var(--slate-200);
  border-top: 3px solid var(--navy-700);
  padding: 14px 18px 12px; border-radius: 4px;
}
.stat-card .big {
  font-size: 42px; font-weight: 700; color: var(--navy-700);
  line-height: 1.0; letter-spacing: -0.035em;
  font-variant-numeric: tabular-nums; margin-bottom: 8px;
}
.stat-card .label { font-size: 11.5px; color: var(--slate-800); font-weight: 500; line-height: 1.38; margin-bottom: 6px; }
.stat-card .source { font-size: 9.5px; color: var(--slate-400); font-style: italic; line-height: 1.3; }

/* Three-windows */
.three-windows { display: grid; grid-template-columns: repeat(3, 1fr); gap: 12px; margin: 8px 0; }
.window-card { border: 1px solid var(--slate-200); background: var(--white); padding: 12px 14px; border-radius: 4px; }
.window-card .head {
  font-size: 10px; font-weight: 600; text-transform: uppercase; letter-spacing: 0.1em;
  margin-bottom: 7px; padding-bottom: 5px; border-bottom: 1px solid var(--slate-200);
}
.window-card.navy .head  { color: var(--navy-700);  border-bottom-color: var(--navy-700); }
.window-card.amber .head { color: var(--amber-700); border-bottom-color: var(--amber-500); }
.window-card.teal .head  { color: var(--teal-700);  border-bottom-color: var(--teal-500); }
.window-card .date { margin-top: 6px; font-size: 9.5px; letter-spacing: 0.05em; color: var(--slate-400); font-variant-numeric: tabular-nums; font-style: italic; }

/* Compare-and-contrast */
.compare { display: grid; grid-template-columns: 1fr 1fr; gap: 12px; margin: 8px 0; }
.compare-card { padding: 12px 16px; border-radius: 4px; }
.compare-card.without { background: #FBEFEF; border: 1px solid #F1D6D6; }
.compare-card.with    { background: #ECF7EF; border: 1px solid #CFE7D6; }
.compare-card h4 { margin: 0 0 6px; font-size: 10.5px; text-transform: uppercase; letter-spacing: 0.08em; font-weight: 600; }
.compare-card.without h4 { color: var(--danger); }
.compare-card.with    h4 { color: var(--success); }
.compare-card ul li::before          { background: var(--danger); }
.compare-card.with ul li::before     { background: var(--success); }

/* Four-pillars */
.four-pillars { display: grid; grid-template-columns: repeat(4, 1fr); gap: 10px; margin: 8px 0; }
.pillar { background: var(--white); border-radius: 4px; overflow: hidden; border: 1px solid var(--slate-200); }
.pillar .head { background: var(--navy-700); color: var(--white); padding: 7px 11px; display: flex; align-items: center; gap: 7px; }
.pillar .head .num { font-size: 9.5px; font-variant-numeric: tabular-nums; letter-spacing: 0.06em; opacity: 0.6; }
.pillar .head .icon { width: 14px; height: 14px; color: var(--white); stroke-width: 2; }
.pillar .head .name { font-weight: 600; font-size: 12.5px; letter-spacing: -0.005em; margin-left: auto; }
.pillar .body { padding: 8px 11px; }

/* Big-date callout + mapping */
.big-date { display: grid; grid-template-columns: 240px 1fr; gap: 16px; margin: 8px 0; }
.date-card { background: var(--amber-500); padding: 18px 18px; border-radius: 4px; }
.date-card .label { font-size: 10px; text-transform: uppercase; letter-spacing: 0.12em; color: var(--navy-900); margin-bottom: 7px; font-weight: 600; }
.date-card .date  { font-size: 30px; color: var(--navy-900); font-weight: 700; line-height: 1.05; letter-spacing: -0.015em; font-variant-numeric: tabular-nums; }
.date-card .sub   { font-size: 10.5px; color: var(--navy-900); margin-top: 8px; line-height: 1.4; }
.mapping { background: var(--white); border: 1px solid var(--slate-200); padding: 8px 14px; border-radius: 4px; }
.mapping .row { display: grid; grid-template-columns: 1fr 14px 1fr; gap: 10px; padding: 3px 0; border-bottom: 1px solid var(--slate-100); font-size: 10.5px; line-height: 1.35; }
.mapping .row:last-child { border-bottom: none; }
.mapping .row .arrow { color: var(--teal-500); text-align: center; font-weight: 600; }

/* Layer-stack */
.layer-stack { display: grid; gap: 3px; margin: 4px 0; }
.layer { display: grid; grid-template-columns: 170px 1fr; gap: 10px; align-items: stretch; background: var(--white); border: 1px solid var(--slate-200); border-radius: 3px; overflow: hidden; }
.layer .label { background: var(--navy-700); color: var(--white); padding: 5px 12px; display: flex; flex-direction: column; justify-content: center; }
.layer .label .num  { font-size: 8.5px; letter-spacing: 0.12em; text-transform: uppercase; opacity: 0.65; font-variant-numeric: tabular-nums; }
.layer .label .name { font-weight: 600; font-size: 11.5px; letter-spacing: -0.005em; margin-top: 1px; }
.layer .body { padding: 5px 12px; font-size: 10px; line-height: 1.4; color: var(--slate-600); align-self: center; }
.layer.cross { background: var(--slate-100); }
.layer.cross .label { background: var(--teal-700); }

/* Arch-pipeline */
.arch-pipeline { display: grid; grid-template-columns: repeat(7, 1fr); gap: 14px; margin: 4px 0 10px; position: relative; }
.arch-stage { background: var(--white); border: 1px solid var(--slate-200); border-top: 2.5px solid var(--navy-700); border-radius: 4px; padding: 7px 5px 7px; text-align: center; display: flex; flex-direction: column; align-items: center; min-height: 72px; position: relative; }
.arch-stage:not(:last-child)::after { content: '→'; position: absolute; right: -14px; top: 50%; transform: translate(50%, -50%); color: var(--teal-500); font-size: 16px; font-weight: 600; width: 14px; text-align: center; line-height: 1; }
.arch-stage .icon { width: 20px; height: 20px; color: var(--navy-700); margin-bottom: 4px; stroke-width: 1.8; }
.arch-stage .num  { font-size: 8.5px; color: var(--slate-400); letter-spacing: 0.1em; font-variant-numeric: tabular-nums; margin-bottom: 2px; }
.arch-stage .name { font-size: 10px; color: var(--slate-950); font-weight: 600; line-height: 1.2; }

/* QA-list (objections + answers) */
.qa-list { display: grid; gap: 7px; margin: 3px 0; }
.qa { display: grid; grid-template-columns: 220px 1fr; gap: 16px; align-items: start; border-bottom: 1px solid var(--slate-200); padding-bottom: 6px; }
.qa:last-child { border-bottom: none; padding-bottom: 0; }
.qa .obj { font-size: 11px; font-weight: 600; color: var(--danger); line-height: 1.36; }
.qa .obj::before { content: 'OBJECTION'; display: block; font-size: 8px; letter-spacing: 0.14em; color: var(--slate-400); font-weight: 600; margin-bottom: 2px; }
.qa .ans { font-size: 10.5px; line-height: 1.42; color: var(--slate-600); }
.qa .ans::before { content: 'ANSWER'; display: block; font-size: 8px; letter-spacing: 0.14em; color: var(--teal-700); font-weight: 600; margin-bottom: 2px; }
```

### 7.6 Fill-screen mode

```css
body.fill-screen .stage { background: var(--slate-50); padding: 0; }
body.fill-screen .stage::before,
body.fill-screen .stage::after { display: none; }
body.fill-screen .frame {
  border-radius: 0 !important;
  box-shadow: none !important;
  transform-origin: center center !important;
}
body.fill-screen .hud      { color: rgba(15,18,28,0.4); }
body.fill-screen .hud .key { border-color: rgba(15,18,28,0.18); background: rgba(15,18,28,0.04); }
body.fill-screen .hud .pos { color: rgba(15,18,28,0.7); }
```

### 7.7 HUD, progress, overview

```css
.hud {
  position: fixed; bottom: 16px; left: 0; right: 0;
  display: flex; justify-content: space-between; align-items: center;
  padding: 0 20px;
  color: rgba(255,255,255,0.45); font-size: 11px;
  letter-spacing: 0.1em; text-transform: uppercase;
  pointer-events: none; z-index: 20;
  font-variant-numeric: tabular-nums;
}
.hud-left  { display: flex; gap: 18px; align-items: center; }
.hud-right { display: flex; gap: 18px; align-items: center; pointer-events: auto; }
.hud .pos  { color: rgba(255,255,255,0.7); font-weight: 600; }
.hud .hint { cursor: help; }
.hud .key  { border: 1px solid rgba(255,255,255,0.2); padding: 3px 7px; border-radius: 3px; font-size: 10px; letter-spacing: 0.06em; background: rgba(255,255,255,0.04); }

.progress {
  position: fixed; top: 0; left: 0; right: 0;
  height: 2px; background: rgba(255,255,255,0.06); z-index: 30;
}
.progress .bar { height: 100%; background: var(--teal-500); transition: width 280ms ease; }

.overview {
  position: fixed; inset: 0; background: rgba(15,18,28,0.96); z-index: 50;
  display: none; padding: 40px 48px; overflow-y: auto;
}
.overview.open { display: block; }
.overview h2 { color: var(--white); font-weight: 600; font-size: 22px; letter-spacing: -0.005em; margin: 0 0 6px; }
.overview p.sub { color: rgba(255,255,255,0.5); font-size: 12px; letter-spacing: 0.08em; text-transform: uppercase; margin: 0 0 28px; }
.overview-grid { display: grid; grid-template-columns: repeat(auto-fill, minmax(220px, 1fr)); gap: 14px; }
.overview-card {
  background: var(--white); border-radius: 6px;
  padding: 14px 16px; cursor: pointer;
  transition: transform 160ms ease, box-shadow 160ms ease;
  border: 1px solid var(--slate-200);
}
.overview-card:hover   { transform: translateY(-2px); box-shadow: 0 8px 24px rgba(0,0,0,0.3); }
.overview-card.current { border: 2px solid var(--teal-500); }
.overview-card.divider { background: linear-gradient(135deg, var(--navy-900) 0%, var(--navy-700) 100%); color: var(--white); }
.overview-card.divider .num   { color: var(--teal-500); }
.overview-card.divider .title { color: var(--white); }
.overview-card .num   { font-size: 10px; letter-spacing: 0.1em; text-transform: uppercase; color: var(--slate-400); font-weight: 600; font-variant-numeric: tabular-nums; margin-bottom: 5px; }
.overview-card .title { color: var(--slate-950); font-weight: 600; font-size: 13px; letter-spacing: -0.005em; line-height: 1.3; }
.overview .close-hint { position: absolute; top: 20px; right: 24px; color: rgba(255,255,255,0.4); font-size: 11px; letter-spacing: 0.1em; text-transform: uppercase; }
```

### 7.8 Print guard

The Web Viewer must remain print-safe even if someone hits Cmd-P on it. The print companion file is the canonical print artefact, but this guard prevents disasters:

```css
@media print {
  body { background: #FFF; }
  .stage::before, .stage::after,
  .hud, .progress, .overview { display: none !important; }
  .stage { position: static; display: block; background: #FFF; }
  .frame { box-shadow: none; width: auto; height: auto; transform: none !important; }
  .slide { display: flex !important; page-break-after: always; }
}
```

### 7.9 Box weight & elevation

A 1px hairline reads thin at slide-presentation distance — the 1280×720 frame, scaled up to a projector or a 27" display, makes the border look weaker than it should. From 1.2 onward, every card-class border is set ~30% heavier than the 1.0 baseline, and every card carries a subtle two-stop drop shadow.

**Border weights (1.2 baseline — multiply 1.0 values by 1.3).**

| Role | 1.0 value | 1.2 value |
|------|-----------|-----------|
| Hairline (card outline, header rule, table row rule) | `1px` | **`1.3px`** |
| Subtle accent (req-card top, layer rule) | `2px` | **`2.6px`** |
| Standard accent bar (stat-card top, buyer-card top, lane-row left) | `3px` | **`3.9px`** |
| Strong accent (callout left, four-pillar head) | `4px` | **`5.2px`** |
| Maximal accent (rare — {{COMPANY_NAME}}-lane mid emphasis) | `5px` | **`6.5px`** |

The scale-up never crosses the `border-radius` boundary — corner rounding stays at `3px–6px` per pattern. `border-collapse` and `border-spacing` are also left alone.

**Drop shadow rule.** A single rule applied to every card-class element:

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
  box-shadow: 0 1px 2.5px rgba(15, 20, 34, 0.08),
              0 0.5px 1px rgba(15, 20, 34, 0.06);
}
```

The two stops give the visual signature of a CSS elevation token (a tight ambient + a soft drop) without crossing into the "Material card" register that is off-brand. Opacity is kept at 0.06–0.08 — visible at presenter distance, invisible at desktop reading distance unless you're looking for it.

**Authoring note.** If you add a new card-class pattern (e.g. a new pricing card on a future slide), append its selector to the box-shadow rule. The simplest way to keep the catalogue consistent is to grep the file for `box-shadow:` and confirm the new pattern is in the same list.

**Do not.** Add a drop shadow to `.bottom-callout`, `.compare-callout`, `.cycle-callout` or any callout-bar element. Callouts are deliberately flat — the colour does the work.

---

## 8. JavaScript essentials

Single IIFE. Drop into the `<script>` block at the foot of the body.

```javascript
(function() {
  const stage = document.getElementById('stage');
  const frame = document.getElementById('frame');
  const overview = document.getElementById('overview');
  const overviewGrid = document.getElementById('overview-grid');
  const progressBar = document.getElementById('progress-bar');
  const titleDisplay = document.getElementById('title-display');
  const posDisplay = document.getElementById('pos-display');

  const slides = Array.from(document.querySelectorAll('.slide'));
  const total = slides.length;
  let idx = 0;

  // Build overview thumbnails
  slides.forEach((s, i) => {
    const card = document.createElement('div');
    card.className = 'overview-card' + (s.dataset.kind === 'divider' ? ' divider' : '');
    card.innerHTML =
      `<div class="num">${String(i + 1).padStart(2, '0')} / ${total}</div>` +
      `<div class="title">${s.dataset.title || ''}</div>`;
    card.addEventListener('click', () => { go(i); toggleOverview(false); });
    overviewGrid.appendChild(card);
  });
  const overviewCards = Array.from(overviewGrid.children);

  function go(n) {
    if (n === idx) return;
    slides[idx].classList.remove('active');
    overviewCards[idx]?.classList.remove('current');
    idx = n;
    slides[idx].classList.add('active');
    overviewCards[idx]?.classList.add('current');
    progressBar.style.width = (idx / (total - 1) * 100) + '%';
    titleDisplay.textContent = slides[idx].getAttribute('data-title') || '';
    posDisplay.textContent = ` · ${String(idx + 1).padStart(2, '0')} of ${String(total).padStart(2, '0')}`;
    if (history.replaceState) history.replaceState(null, '', '#slide-' + idx);
  }
  function next() { go(Math.min(idx + 1, total - 1)); }
  function prev() { go(Math.max(idx - 1, 0)); }
  function toggleOverview(force) {
    const willOpen = force === undefined ? !overview.classList.contains('open') : force;
    overview.classList.toggle('open', willOpen);
  }
  function setFillScreen(on) { document.body.classList.toggle('fill-screen', on); fit(); }
  function toggleFullscreen(fillScreen) {
    const inFs = !!document.fullscreenElement;
    const inFill = document.body.classList.contains('fill-screen');
    if (!inFs) {
      (document.documentElement.requestFullscreen || (() => {})).call(document.documentElement);
      setFillScreen(!!fillScreen);
    } else if ((!!fillScreen) === inFill) {
      (document.exitFullscreen || (() => {})).call(document);
      setFillScreen(false);
    } else {
      setFillScreen(!!fillScreen);
    }
  }
  document.addEventListener('fullscreenchange', () => {
    if (!document.fullscreenElement) setFillScreen(false);
  });
  function openPrint() {
    // Replace with the actual print file name for this deliverable
    window.open('{{Document name}}-print.html', '_blank');
  }
  function showHelp() {
    alert('{{Document name}} · Keyboard\n\n' +
      '→  Space  PageDown   Next slide\n' +
      '←  Backspace  PageUp Previous slide\n' +
      'Home  /  End          First / last slide\n' +
      'O  /  Esc             Toggle overview\n' +
      'f                     Fullscreen — fit\n' +
      '⇧F                    Fullscreen — fill\n' +
      'Esc                   Exit fullscreen\n' +
      'P                     Open print edition\n' +
      '?                     This help');
  }

  document.addEventListener('keydown', (e) => {
    if (e.metaKey || e.ctrlKey || e.altKey) return;
    const k = e.key;
    if (overview.classList.contains('open')) {
      if (k === 'Escape' || k === 'o' || k === 'O') { toggleOverview(false); e.preventDefault(); }
      return;
    }
    if      (k === 'ArrowRight' || k === ' ' || k === 'PageDown')      { next(); e.preventDefault(); }
    else if (k === 'ArrowLeft'  || k === 'Backspace' || k === 'PageUp'){ prev(); e.preventDefault(); }
    else if (k === 'Home')    { go(0); e.preventDefault(); }
    else if (k === 'End')     { go(total - 1); e.preventDefault(); }
    else if (k === 'o' || k === 'O') { toggleOverview(); e.preventDefault(); }
    else if (k === 'f')       { toggleFullscreen(false); e.preventDefault(); }
    else if (k === 'F')       { toggleFullscreen(true);  e.preventDefault(); }
    else if (k === 'p' || k === 'P') { openPrint(); e.preventDefault(); }
    else if (k === '?')       { showHelp(); e.preventDefault(); }
    else if (k === 'Escape')  { toggleOverview(false); }
  });

  stage.addEventListener('click', (e) => {
    if (e.target.closest('.frame') || e.target.closest('.hud') || e.target.closest('.overview')) return;
    const x = e.clientX, w = window.innerWidth;
    if (x < w * 0.25) prev(); else if (x > w * 0.75) next();
  });

  // Deep link
  const m = (location.hash || '').match(/slide-(\d+)/);
  if (m) { const n = Math.min(total - 1, Math.max(0, parseInt(m[1], 10))); if (n !== 0) go(n); }

  // Initial state
  progressBar.style.width = (idx / (total - 1) * 100) + '%';
  titleDisplay.textContent = slides[idx].getAttribute('data-title') || '';
  posDisplay.textContent = ` · ${String(idx + 1).padStart(2, '0')} of ${String(total).padStart(2, '0')}`;
  overviewCards[idx]?.classList.add('current');

  function fit() {
    const isFill = document.body.classList.contains('fill-screen');
    if (isFill) {
      frame.style.transform = `scale(${window.innerWidth / 1280}, ${window.innerHeight / 720})`;
    } else {
      const vw = window.innerWidth - 64;
      const vh = window.innerHeight - 64 - 40;
      const s = Math.min(vw / 1280, vh / 720, 1.4);
      frame.style.transform = `scale(${s})`;
    }
  }
  window.addEventListener('resize', fit);
  fit();
})();
```

---

## 9. Slide templates

Concrete templates for the three slide kinds. Adapt copy; keep structure.

### 9.1 Cover

```html
<section class="slide cover active" data-title="Cover" data-kind="cover">
  <header class="slide-head">
    <span>{{Document name}}</span>
    <span>Edition {{X.Y}} · {{YYYY-MM-DD}}</span>
  </header>
  <div class="slide-body">
    <div class="cover-center">
      <span class="tag">{{Short positioning chip}}</span>
      <h1>{{Document<br>title.}}</h1>
      <p class="lede">{{One-sentence framing — what this document is and for whom.}}</p>
    </div>
    <div class="cover-meta-row">
      <div>Document<span>{{Document name}}</span></div>
      <div>Edition<span>{{X.Y · YYYY-MM-DD}}</span></div>
      <div>Owner<span>{{Name}}<br>{{email}}</span></div>
      <div>Audience<span>{{Audience}}</span></div>
    </div>
  </div>
  <footer class="slide-foot">
    <span>→ to begin · ? for keyboard help</span>
    <span>Cover</span>
  </footer>
</section>
```

### 9.2 Part divider

```html
<section class="slide part-divider" data-title="Part I · {{Part name}}" data-kind="divider">
  <header class="slide-head"><span>Part I</span><span class="brand">{{COMPANY_NAME}}</span></header>
  <div class="slide-body">
    <div class="part-number">Part I · {{Part name}}</div>
    <h1>{{Part action title.}}</h1>
    <p class="part-lede">{{One-paragraph framing for this part.}}</p>
    <div class="part-chips">
      <span class="chip">Slides {{N–M}}</span>
      <span class="chip">{{Topic chips · MECE · comma separated}}</span>
    </div>
  </div>
  <footer class="slide-foot"><span>Part I divider</span><span>—</span></footer>
</section>
```

### 9.3 Content slide

```html
<section class="slide" data-title="{{NN}} · {{Action title}}" data-kind="content">
  <header class="slide-head">
    <span>Part {{Roman}} · {{Part name}}</span>
    <span class="brand">{{COMPANY_NAME}}</span>
  </header>
  <div class="slide-body">
    <h1>{{Action title — a claim, not a topic.}}</h1>
    <div class="accent-bar"></div>
    <p class="subtitle">{{Optional subtitle that earns the claim.}}</p>

    <!-- Choose ONE primary pattern: three-stat, three-windows, four-pillars,
         compare-and-contrast, big-date callout + mapping, layer-stack,
         arch-pipeline, qa-list, or a simple .cols layout. -->

    <div class="bottom-callout">
      {{Italicised one-liner that compresses the slide's claim. Never repeats the title.}}
    </div>
  </div>
  <footer class="slide-foot">
    <span>{{NN}} · {{Action title}}</span>
    <span>Slide {{N}} of {{Total}}</span>
  </footer>
</section>
```

---

## 10. Authoring discipline

The Web Viewer is just a vessel. The discipline that makes it land is the brand voice. Quick reference for slide-authoring:

1. **Every slide title is an action title** — a claim, not a topic. Sentence case. No end punctuation. The reader who only sees the title knows what the slide argues.
2. **Every number carries an inline source.** Stat cards have a `.source` line; tables cite in caption; in-line claims add `(Source · Year)`.
3. **Bottom-callout never repeats the title.** It compresses the slide into a single italicised line.
4. **MECE structure.** Three-stat = three. Three-windows = three. Four-pillars = four. If you cannot make the set fit, the structural choice is wrong.
5. **Lead with Navy. Accent with Teal. Reserve Amber for compliance and deadlines.** No more than three colour roles on a single slide.
6. **No banned phrases.** No *transformative · magical · seamless · revolutionise · just works · vendor-neutral · single source of truth · AI-native · 10x · cutting-edge · empower · democratise · unleash · supercharge*. See brand guide §2.5.
7. **Verbal signature recurs.** If the brand has a recurring verbal signature — a set phrase or cadence that should echo across decks — apply it consistently. Record the brand's signature as `{{BRAND_VOICE_SIGNATURE}}` in `brand.md`.
8. **State non-goals.** What the product does not do appears as visibly as what it does.
9. **Cover first; part dividers between sections; content slides numbered continuously.** Cover and dividers are not numbered.
10. **Action title font: Inter 28px / 700 / Navy 700.** Resist the temptation to scale up.

---

## 11. Common pitfalls

**Slides overflowing the frame.** Symptoms: scrollbar appears inside the frame; content clipped at the bottom. Causes: too many bullets, too-large type, too-thick padding. Fixes (in order): reduce list density to ≤5 items, drop type by 1px, tighten section padding, split into two slides.

**Output token limit hit.** A 50+ slide deck plus full CSS is large. Write in pieces: skeleton + first 10–15 slides in a `Write` call, then `Edit` operations appending blocks of 10–15 slides before the closing `</div></div>` tags.

**Fullscreen fill positioning bug.** Symptom: in `Shift+F` mode, content appears in a corner instead of centred. Cause: `transform-origin` defaults to `top left`. Fix: keep `transform-origin: center center !important;` under `body.fill-screen .frame`.

**The chord glyph in the brand mark looks wrong on dark.** Symptom: dark mark on dark background invisible. Fix: cover and part-divider override the `.brand::before` border to white; do this for any dark slide.

**Print HTML diverges.** Symptom: print version misses a slide that's in the viewer. Cause: someone edited one file but not the other. Fix: re-author from the markdown source if it exists; otherwise compare slide counts (`grep -c '<section class="slide"' file.html`) and reconcile.

**Wrong file linked from `P` key.** Symptom: `P` opens the wrong print file or 404s. Cause: `openPrint()` hardcodes the print filename. Fix: update the filename in the `openPrint` function to match the actual print file's name.

**Deep links broken.** Symptom: `#slide-N` doesn't navigate to slide N. Cause: someone reordered slides without checking. Fix: use `data-title`-based anchors instead if order is volatile; otherwise just be deliberate about renumbering.

**Section padding mismatched.** Symptom: cover/divider slides look cramped or content slides look wasteful. Fix: cover and dividers use `padding: 60px 80px`; content slides use `padding: 36px 64px 32px`.

---

## 12. Sister format — Print HTML

The print HTML companion is the deck's A4 landscape edition. It is produced from the same content but optimised for paper and PDF export.

Key differences:

| Aspect | Web Viewer | Print HTML |
|--------|------------|------------|
| Page size | 1280×720 frame in viewport | A4 landscape (297mm × 210mm) |
| Navigation | Keyboard + click | Scroll (or paginated print preview) |
| Pagination | Per-section CSS animation | `@page { size: A4 landscape; margin: 0; }` + `break-after: page` |
| Background | Slate-950 stage, frame on top | White page, content full-bleed |
| HUD | Visible | Absent |
| Print bleed | Not relevant | Cover and dividers bleed (`box-shadow: 0 0 0 5mm` or `::before { inset: -5mm; }`) |

The print file is its own deliverable, not a stylesheet variant. The `@media print` block in the Web Viewer is a defensive fallback; the print HTML is the canonical print artefact.

When updating a deck:

1. Edit the markdown source if it exists.
2. Regenerate both HTMLs from the source — do not edit them independently.
3. Verify slide counts match: `grep -c '<section class="slide"' Web-Viewer.html` and `grep -c '<section' Print.html` (counting cover + dividers + content slides; the section selector may differ slightly between files).
4. Update version and date in cover, footers, and meta.

---

## 13. Quick start

To produce a new Web Viewer for an existing markdown source:

1. Read the markdown to establish slide breakdown and content patterns needed.
2. Copy the **HTML skeleton (§6)** into a new file at `<topic>-<version>.html`.
3. Paste the **full CSS (§7)** into the `<style>` block.
4. Paste the **full JavaScript (§8)** into the closing `<script>` block.
5. Build the cover slide using **§9.1**.
6. For each part, build a part-divider slide using **§9.2** and the content slides under it using **§9.3**.
7. Update the `openPrint()` function (§8) with the actual print file name.
8. Update the `showHelp()` document title in the alert (§8) with the actual document name.
9. Verify slide counts and `data-title` attributes are populated.
10. Produce the print HTML companion in parallel, sharing all content.

Done. Open the file in a browser and the deck navigates.

---

## 14. Responsiveness — desktop, tablet, phone

The 1.0 viewer rendered the 1280×720 frame uniformly across all devices and listened only to keyboard and outer-stage clicks. On iPad with a hardware keyboard, keypresses sometimes did not reach the document; on iPhone, the frame scaled down to roughly 30% and became unreadable. 1.1 keeps the design intact at desktop and adds three layers of responsiveness on top.

### 14.1 Layer A — touch-and-keyboard parity

Three additions make the viewer work for every input mode without changing the slide design.

**Focus-trap input.** A 1×1 px hidden `<input class="kbd-trap" tabindex="-1" readonly>` lives at the top of the body. The viewer focuses it on load, on every slide change, on overview close, and on `visibilitychange`. iOS Safari only fires hardware-keyboard `keydown` events on focused elements; without the trap, a touch anywhere can shift focus to a non-focused element and silently drop key events. The trap is `font-size: 16px` to prevent iOS' "tap-to-zoom" behaviour, `opacity: 0`, and `pointer-events: none`.

**Touch gestures.** The `.stage` element handles `touchstart` / `touchend` on the non-frame area:

| Gesture | Action |
|---|---|
| Swipe left → | Next slide |
| Swipe right → | Previous slide |
| Tap on left third | Previous slide |
| Tap on right third | Next slide |
| Tap on middle third | Toggle touch HUD |
| Two-finger tap | Toggle overview |

Swipe threshold is 50px with `dx > dy * 1.4` to distinguish horizontal intent from a vertical scroll attempt. Touch listeners are `{ passive: true }` so they never block scrolling.

**Touch HUD.** A pill-shaped bottom bar (`.touch-hud`) with five 44×36 px tap targets — prev, position pill, next, overview, fullscreen — appears whenever the user is interacting and auto-hides after 3.2 s of inactivity. The HUD shows automatically on touch devices (detected via `matchMedia('(pointer: coarse)')` and `ontouchstart`); the desktop keyboard-hint strip is hidden when the touch HUD is shown.

### 14.2 Layer B — responsive frame scaling

The frame still targets 1280×720 on tablet, but with adjusted margins and dynamic-viewport math:

- `vh` is replaced with `window.innerHeight` (which tracks the dynamic viewport on iOS Safari without a `dvh` polyfill in JS).
- At viewport width ≤ 1024 px, the outer click zones (`.stage::before/after`) hide — touch handles navigation — and frame margins drop from 64×104 px (desktop) to 20×70 px (tablet) so the frame fills more of the screen.
- `orientationchange` triggers a delayed `fit()` call (250 ms) because iOS Safari fires the event before window dimensions stabilise.
- `viewport-fit=cover` is set in the `<meta>` so notched devices render correctly; HUDs respect `env(safe-area-inset-bottom)`.

### 14.3 Layer C — phone stack mode

At viewport width ≤ 640 px the layout fundamentally changes. The fixed-frame transform is dropped; slides become a horizontally scroll-snapped strip, each pane is full-viewport, content scrolls vertically inside.

**Container.** `.frame` becomes a flex row with `overflow-x: auto; scroll-snap-type: x mandatory; overscroll-behavior-x: contain;`.

**Pane.** Each `.slide` is `flex: 0 0 100vw; width: 100vw; height: 100dvh; overflow-y: auto; scroll-snap-align: start; scroll-snap-stop: always;`. The desktop "only `.active` visible" rule is overridden; all slides render simultaneously.

**Typography reflows.** Body text bumps from 11–12 px to 14–15 px; H1 from 28 px to 22 px (slide), 54 px (cover), 44 px (part divider). Subtitle, callouts, list items all get phone-readable sizes.

**Content patterns collapse.** Every multi-column grid pattern — `three-stat`, `three-windows`, `four-pillars`, `compare`, `levels-band`, `levels-ladder`, `flywheel`, `buyer-grid`, `discovery-grid`, `req-grid`, `arch-blocks`, `big-date`, `cols`, `lane-row` — converts to `grid-template-columns: 1fr` and stacks vertically.

**Arch-pipeline and flywheel arrows** rotate from `→` to `↓` and reposition with `top: 100%; transform: translateY(-50%)`.

**Tables collapse to cards.** `.method-table` and `.moat-row` use `display: block`; the original `<th>` row is hidden via `display: none` on `thead`; each cell shows its label via a `data-label="…"` attribute and `td::before { content: attr(data-label); }`. Authors must populate `data-label` on every `<td>` (except the first column, which already serves as the card header).

**Position tracking.** An `IntersectionObserver` watches every slide with `root: frame` and `threshold: [0.5, 0.75]`. When the user swipes, the slide with the highest intersection ratio becomes the current index; the HUD position updates accordingly. JS-initiated navigation (`go(n)`) sets a 600 ms `suppressScrollObserver` flag so observer callbacks don't fight the programmatic scroll.

**Mode switching.** A `matchMedia('(max-width: 640px)')` listener handles rotation and orientation changes. Entering phone mode: clear all `.active` classes (let all slides render), set up the observer, scroll to the current index. Leaving phone mode: disconnect the observer, restore the single-`.active` model, call `fit()`, refocus the keyboard trap.

### 14.4 Author requirements for v1.1

When authoring or modifying a Web Viewer at 1.1:

1. **`data-label` on every `<td>`** in any table that should collapse to a card on phone (`.method-table`, `.moat-row`). The first column does not need one — it is treated as the card header. Without `data-label`, mobile users see unlabelled values.
2. **Avoid hard-coded pixel widths** in slide content; use percentages or `1fr` so the phone reflow keeps the layout usable.
3. **Test the focus trap** on iPad Safari with a hardware keyboard — type `?` to confirm the help dialog opens. If it doesn't, something in the page is stealing focus on touch.
4. **No `localStorage`/`sessionStorage`** — already a brand-wide rule, but matters here because phone reflow re-runs on rotate.
5. **Update the print-file link** in `openPrint()` to match the actual `-print.html` companion (still required from 1.0).

### 14.5 Browser support and graceful fallback

- `scroll-snap-type: x mandatory` — supported in Safari 11+, Chrome 69+, Firefox 68+. iOS Safari supports it natively from iOS 11.
- `100dvh` — Safari 15.4+, Chrome 108+. Older browsers fall back to `100vh`, accepting the address-bar gap.
- `env(safe-area-inset-*)` — universal on modern WebKit/Blink/Gecko.
- `IntersectionObserver` — universal since ~2019.
- `viewport-fit=cover` — required for the inset env values to register on notched iPhones.

If the browser is too old for `scroll-snap`, the phone mode degrades to a free horizontal scroll without snap — still usable, just less polished. The focus trap is a no-op on browsers that ignore programmatic input focus.

### 14.6 iPad robustness (1.3)

iPad is the awkward middle: too big for the 1.1 phone stack mode (>640px), too touch-first for desktop click zones, and frequently opened in non-Safari contexts (Messages attachment preview, Files-app QuickLook, in-app browsers) where JS event handlers fire unreliably or not at all.

**1.3 changes the trigger for stack mode from a viewport-width check to a touch-capability check.** The breakpoint becomes:

```css
@media (max-width: 640px),
       (any-pointer: coarse) and (max-width: 1400px) {
  /* phone / tablet stack mode */
}
```

```js
const phoneMq = window.matchMedia(
  '(max-width: 640px), (any-pointer: coarse) and (max-width: 1400px)'
);
```

That trigger fires for:
- Every iPhone (≤640px in portrait covers them all).
- Every iPad in every orientation — the `(any-pointer: coarse)` half matches because the touchscreen is always present, even when a Magic Keyboard with trackpad is attached.
- Any touchscreen laptop under ~1400px wide.

It does **not** fire for desktops, large external touchscreens, or laptops without a touchscreen.

**Why this is robust.** Stack mode uses native CSS scroll-snap for slide navigation:

```css
.frame {
  display: flex; flex-direction: row;
  overflow-x: auto; overflow-y: hidden;
  scroll-snap-type: x mandatory;
}
.slide { flex: 0 0 100vw; scroll-snap-align: start; }
```

The browser's compositor handles the swipe directly — no JS event handler involved. So swipe navigation works in every iPad context: Safari, Files-app preview, Messages attachment, in-app WKWebView, QuickLook. JS observes the scroll position via `IntersectionObserver` to update the HUD, but navigation itself never depends on JS firing.

**The trade-off.** Tablets get the same reflowed content layout as phones: single-column grids, larger typography, table-to-card collapse. That's a deliberate aesthetic compromise — the slide layout is built for a 1280×720 frame at desktop, and naively shrinking it to a 768-px portrait pane would produce 5-px body text. Reflow is the right answer when the goal is "works reliably and reads well".

**Belt-and-braces JS handlers retained.** Keydown is still bound on `document` + `body` + `kbd-trap` with a per-event dedupe flag, and Pointer Events still provide a parallel touch fallback. Those are no longer the primary navigation mechanism — they're the secondary layer that handles hardware keys, two-finger overview tap, etc. The primary mechanism (scroll-snap) is bulletproof.

**Touch on iPad — four mechanisms in parallel.**

Touch events alone aren't enough on iPad because:
- `{ passive: true }` listeners can't `preventDefault()`, so iOS sometimes interprets a tap as a scroll and drops the `touchend`.
- In Messages-attachment / QuickLook contexts, touch events may dispatch on the document but not bubble up to a deeply-nested handler.
- A focused `<input readonly>` doesn't always bubble keypresses on certain iPadOS versions.

The fix is to listen on multiple layers and dedupe per-event:

1. **Touch events on `.stage` AND `document`.** Both fire; a `e.__cogTouched` flag set by whichever fires first prevents double-handling.
2. **Pointer Events on `document`.** A unified `pointerdown`/`pointerup` pair that captures touch + pencil + trackpad + mouse, gated to only fire on coarse pointers or non-mouse pointer types.
3. **Lowered thresholds.** Swipe minimum 30px (was 50px), axis-ratio 1.2× (was 1.4×), maximum duration 1000ms (was 800ms). Tap movement tolerance 16px (was 12px), duration 500ms (was 350ms).
4. **Refocus on every gesture.** Every `touchstart` / `pointerdown` calls `refocusTrap()` — touch is a user activation, so `.focus()` always succeeds even when programmatic focus on load was silently rejected.

**Keyboard on iPad — three listeners, deduped.**

```js
// Bind to document, body, AND the kbd-trap input. iOS sometimes delivers
// keydowns only to the focused element and sometimes only to document;
// binding to all three ensures the handler always sees them. The
// `e.__cogHandled` flag prevents double-fire when the event does bubble.
document.addEventListener('keydown', dedupedHandleKey);
document.body.addEventListener('keydown', dedupedHandleKey);
kbdTrap.addEventListener('keydown', dedupedHandleKey);
```

**Kbd-trap input hardened with iOS-specific attributes.**

```html
<input class="kbd-trap" id="kbd-trap"
       aria-hidden="true" tabindex="-1" readonly
       inputmode="none"
       autocapitalize="off" autocomplete="off"
       autocorrect="off" spellcheck="false">
```

`inputmode="none"` prevents the soft keyboard from appearing even briefly. `autocapitalize`/`autocomplete`/`autocorrect`/`spellcheck` off prevents iOS from attaching its assist UI to the input.

**Inline SVG defensiveness.**

If the page embeds `<symbol>` blocks at the top of `<body>` (the {{COMPANY_NAME}} brand mark from 1.2 does this — see §7.9), the wrapping `<svg>` must use:

```html
<svg style="position:fixed;top:0;left:-9999px;width:0;height:0;
            overflow:hidden;pointer-events:none"
     aria-hidden="true" focusable="false">
  <defs>
    <symbol id="..."> ... </symbol>
  </defs>
</svg>
```

The combination of `position: fixed`, off-viewport `left`, zero dimensions, and explicit `pointer-events: none` guarantees the block can never:
- Intercept touches at the start of body
- Receive focus during keyboard navigation
- Affect the hit-test of any other element

`position: absolute` without `top`/`left` (an earlier convention) caused intermittent touch-capture issues on iPadOS 17 — pin the block off-viewport instead.

**What still has to be true.**

- The host context must allow JavaScript execution. QuickLook on iPadOS 16 and earlier was inconsistent here; iPadOS 17+ runs JS in attachment previews reliably.
- The host context must support Pointer Events. iPadOS Safari has done so since 13.0; only ancient WKWebView contexts are missing it.
- Hardware keyboards must be paired and active. iPadOS shows them in the Settings → General → Keyboard list when present.

### 14.7 What the viewer deliberately does not do

- **No PWA "Add to Home Screen" treatment.** Decision: keep the deck a single linkable artefact, not an installable app.
- **No wake-lock during fullscreen.** Out of scope; iOS Safari support is shaky and the deck is short.
- **No pinch-to-zoom for overview.** Two-finger tap toggles overview; pinch is intentionally not used because iOS Safari intercepts it for page zoom in some cases.
- **No keyboard shortcuts via on-screen keyboard.** Soft keyboards on phones don't have arrow keys; the touch HUD covers everything.

---

## Changelog

- **1.3 — 2026-05-27.** §14.6 — iPad robustness. **Primary fix:** changed the stack-mode trigger from `max-width: 640px` to `max-width: 640px OR (any-pointer: coarse) AND max-width: 1400px`, so every iPad (in every orientation, with or without keyboard) uses native CSS scroll-snap for navigation instead of JS event handlers. Swipe now works in QuickLook, Messages preview, and Files-app preview where JS events are sandboxed. **Secondary fixes (now belt-and-braces only):** keydown bound on document + body + kbd-trap with per-event dedupe; Pointer Events as touch fallback; thresholds widened (swipe 50→30 px, tap 12→16 px); kbd-trap input hardened with `inputmode="none"` and assist-UI off. Inline `<symbol>` blocks must now use `position: fixed; left: -9999px; pointer-events: none` (previously `position: absolute` without offsets, which intermittently intercepted touches on iPadOS 17). Trade-off: tablets now get the same single-column reflowed content layout as phones — the right answer when the alternative is "works on iPhone, broken on iPad".
- **1.2 — 2026-05-27.** Added §7.9 — box weight & elevation. Card-class borders bumped ~30% (1px → 1.3px hairline, etc.) and a two-stop drop shadow applied to every card pattern. Callout bars stay flat. Author requirement: when adding a new card-class pattern, extend the box-shadow rule's selector list.
- **1.1 — 2026-05-22.** Added §14 — responsiveness for tablet and phone. Touch gestures, focus trap for iPad hardware keyboards, phone stack mode with scroll-snap and reflowed content. Author requirements: `data-label` attributes on tables that collapse, no fixed-pixel layout widths.
- **1.0 — 2026-05-18.** Initial specification. Extracted from the reference brand-guide implementation. Reflects the format as shipped across several early decks and memos.
