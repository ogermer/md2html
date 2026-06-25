---
name: create-html-from-md
description: |
  Convert a markdown deck source into the two sibling HTML renderings — the
  on-screen Web Viewer and the A4-landscape Print HTML. Use this skill whenever
  the user has a markdown file structured as numbered slides ("# Slide N · …")
  and wants the HTML versions generated or refreshed. Also use it when the user
  says "build the deck", "regenerate the HTML", "create the print PDF source",
  "convert the markdown to slides", "make a webview", "make a print edition",
  or asks for the screen / print version of an existing markdown deck. The
  skill bundles a complete, configurable visual system (palette, typography, SVG logos,
  responsive layout, drop-shadowed cards) so the output is on-brand without any
  external lookups. Use it even if the user only asks for one of the two
  formats — the skill prompts which to build.
---

# create-html-from-md

## What this skill does

Takes a markdown deck source (one file, structured as a sequence of `# Slide N · Title` blocks with named content patterns) and produces one or both of:

- **Web Viewer HTML** — on-screen, navigable, scroll-snap on phone & tablet (iPad), drop-shadowed cards, keyboard + touch nav. Output name `<base>.html`.
- **Print HTML** — A4 landscape, one slide per page, drop-shadowed cards, optimised for the browser print dialog. Output name `<base>-print.html`.

Both files are **fully standalone** — no external CSS, no external fonts loaded from the network at runtime (Google Fonts pre-link is included so it works online, but the page is functional offline), and the brand logo is embedded inline as an SVG `<symbol>` block.

## When to use

Trigger this skill whenever the conversation involves turning a markdown deck source into HTML — including phrasings like:

- "Build the webview from `…/deck.md`"
- "Generate the print HTML"
- "Create the HTML versions of this deck"
- "Regenerate the HTML for v1.3" / "refresh both HTMLs"
- "Convert the markdown to slides"
- "Make a print PDF from this deck"

The skill is not for: PowerPoint generation (use the canonical `build_pptx.js` pattern instead), single-slide screenshots, or non-deck markdown (essays, reports, READMEs).

## Workflow

Follow these steps in order. Don't skip the format prompt — the user often only wants one of the two HTML files refreshed, and building both wastes time.

### Step 1 · Always ask which format(s) to build

Use `AskUserQuestion` (or, if unavailable, ask inline) with a single question:

> **Which format(s) should I build?**
> - Both (Recommended)
> - Webview only
> - Print only

Only skip the question if the user has already said unambiguously in the same turn (e.g., "build me the print PDF" → print only; "regenerate the webview" → webview only).

### Step 1b · Ask whether to add a confidentiality footer

Single-recipient pitches, customer proposals, and internal-only reviews carry a centred `CONFIDENTIAL — [NAME] [VERSION]` strap in every slide's footer. Public marketing collateral and conference talks do not. The skill should decide as follows:

- **If the markdown's title or filename names a specific recipient** (e.g. `client-pitch-v1.3.md`, `board-proposal.md`), assume yes and propose the strap text. Confirm with the user only if the inferred `[NAME]` or `[VERSION]` is ambiguous.
- **If the markdown has no specific audience cue**, ask:

  > **Add a CONFIDENTIAL footer to every slide?**
  > - Yes — `CONFIDENTIAL — {NAME} {VERSION}` (recommended for single-recipient decks)
  > - No — keep the standard two-column footer

  When the user says yes, derive `{NAME}` (audience in upper case) and `{VERSION}` (e.g. `VERSION 1.3`) from the markdown's title and edition. See `brand.md` § Confidentiality footer for the full template.

### Step 2 · Read the brand configuration

Read `brand.md` (lives next to this SKILL.md). It defines the palette, typography, voice rules, logo paths, file-naming convention, and current edition metadata. The brand file is the single rebranding seam — if it ever changes, the entire skill output changes.

### Step 3 · Read the markdown source

The user will name a path (or it will be implied from context, e.g. an attached file or the most recently edited deck). Read the full file.

Expected structure:
- A leading metadata block (Edition / Date / Presenter).
- An optional "How to read this deck" block (agenda table).
- A sequence of `# Slide N · <Action title>` headings.
- Optional `# Part <Roman> · <Name>` part dividers between groups of slides.
- The final slide is usually `# Slide M · Sources`.

Inside each slide, the markdown uses named pattern markers (italic or bold) that tell you which HTML pattern to assemble:

| Markdown marker | HTML pattern |
|---|---|
| `**Action title.** *…*` | `<h1>` + accent bar |
| `**Subtitle.** …` | `<p class="subtitle">` |
| `**Three-stat.**` | `.three-stat` with three `.stat-card` |
| `**Three-windows pattern.**` | `.three-windows` with three `.window-card` |
| `**Compare-and-contrast.**` | `.compare` with two `.compare-card` |
| `**Two columns.**` | `.cols` with two `.col` |
| `**Four-pillar pattern.**` | `.four-pillars` with four `.pillar` |
| `**Pipeline — seven stages with arrows.**` | `.arch-pipeline` |
| `**Three layers** / `**[icon] Agent layer.**` etc. | `.lane-stack` with `.lane-row` |
| `**Date callout** + **Why…**` | `.big-date` + `.deferral-card` (or `.mapping`) |
| `**Methodology table…**` | `.method-table` |
| `**Bottom callout, teal/amber/navy.** *…*` | `.bottom-callout teal/amber/navy` |
| Objections list of `*Objection — "…"*` + `**Answer.**` pairs | `.qa-list` with `.qa` entries |

When in doubt, read `references/patterns.md` for the full catalogue and HTML snippets to copy and customise.

### Step 4 · Assemble the HTML

For each format the user picked, follow the matching skeleton:

- **Webview** — open `references/skeletons.md` § Webview skeleton, copy it, and fill in the slide loop. Then apply the per-pattern snippets from `references/patterns.md`. The skeleton already includes: brand palette CSS custom properties, typography scale, inline SVG `<symbol>` block for the logo (referenced by `<use href="#company-logo-blue">` / `…-white`), responsive scroll-snap for ≤640 px (iPhone) and `(any-pointer: coarse) and (max-width: 1400px)` (iPad), kbd-trap input + keydown handlers, touch HUD, drop shadows on cards, iPad-robust touch + pointer event handlers.

- **Print** — open `references/skeletons.md` § Print skeleton, copy it, fill in the slide loop using mm units and the same patterns adapted for A4 landscape. The skeleton already includes: the `-1 mm` page-height trick, `@page { size: A4 landscape; margin: 0 }`, the mandatory on-screen print-dialog banner, the drop shadow CSS scoped to print, and the `print-color-adjust: exact` rule so shadows survive the print dialog.

For both formats, **embed both logo SVGs** as `<symbol>` definitions in a single hidden `<svg>` at the top of `<body>` so any number of `<use>` references can reuse them without duplicating the path data. Use the snippet in `references/patterns.md` § Logo symbols. The SVG files in `assets/` are the canonical source — when this skill is initialised in a new project or rebranded, copy the new SVGs in and update `brand.md` accordingly.

### Step 5 · Verify before writing

Before declaring done, run these checks:

1. **Slide count balanced.** Number of `# Slide N · …` headings in source = number of `<section class="slide" data-kind="content">` in webview = number of `<section class="page">` minus the cover and dividers in print.
2. **Footer numbering matches.** Every footer reads `Slide N of M` with the correct N and M.
3. **No stale version refs.** No "v1.1" if you are shipping v1.2; the `openPrint()` call in the webview targets the actual print filename.
4. **Brand mark right-aligned.** The SVG logo `<text>` has `text-anchor="end"` with x at the viewBox right edge — otherwise the wordmark sits ~4 px short of the slide-head's right padding.
5. **JS syntax.** Extract the `<script>` block and run `node --check` (or another quick parser) to confirm no typos broke the keydown/touch handlers.

6. **Confidentiality strap consistent.** If the strap was enabled, every footer carries the same `CONFIDENTIAL — [NAME] [VERSION]` text — no half-renamed leftovers from a previous version, no typos in the audience name. Grep `CONFIDENTIAL` and confirm the count equals the total footer count.
7. **Stale template names gone.** Grep for the previous deck's name / dates to make sure all references were updated.

### Step 6 · Write the output files

Save to the same directory as the markdown source, using the `file_naming` rule in `brand.md` (`{base}.html` and `{base}-print.html` by default). Don't write to `outputs/` — these files belong with the source.

### Step 7 · Present and offer follow-ups

Provide computer:// links so the user can open the files. If only one format was built, offer the second as a follow-up ("Want the print version too?"). If both were built and this is a refresh, suggest converting the print HTML to PDF (`soffice --headless --convert-to pdf …` or Chrome's print dialog).

## Files in this skill

```
create-html-from-md/
├── SKILL.md                  ← you are here
├── brand.md                  ← Cognōri palette / type / voice / file naming. Edit this to rebrand.
├── assets/
│   ├── company-wordmark-blue.svg   ← placeholder wordmark (COMPANY_LOGO); replace when branding
│   └── company-wordmark-white.svg  ← placeholder wordmark (COMPANY_LOGO); replace when branding
└── references/
    ├── webview-spec.md       ← canonical Web Viewer format (v1.3, snapshot)
    ├── print-spec.md         ← canonical Printable Slides briefing (v1.1, snapshot)
    ├── skeletons.md          ← drop-in HTML skeletons for both formats
    └── patterns.md           ← per-pattern HTML snippets (three-stat, three-windows, …)
```

The two `*-spec.md` files are snapshots from the canonical brand-guide — the skill is intentionally self-contained, so it works even if the brand-guide files move or change. If the canonical spec is updated upstream, refresh the snapshots in this skill directly.

## Pitfalls — read this before you ship

These are the failure modes that bite every time. Each one was learned from a real broken deck.

- **CSS specificity bug on the brand mark.** The `.slide-head .brand svg.brand-svg { display: block }` rule has higher specificity than `.slide-head .brand .white { display: none }`. If you write the rule that way, both blue and white logos render side-by-side. Always write `svg.brand-svg.blue { display: block }` and `svg.brand-svg.white { display: none }` so specificity matches.

- **`text-anchor="end"` on the logo wordmark.** The Inkscape-authored SVG has whitespace on the right of the wordmark inside its viewBox. Without anchoring the text element to the viewBox right edge, the wordmark sits visibly short of the slide-head's right padding. The snippet in `patterns.md` § Logo symbols has this baked in.

- **iPad uses scroll-snap, not JS handlers.** Set the stack-mode media query to `(max-width: 640px), (any-pointer: coarse) and (max-width: 1400px)`. iPad in QuickLook / Messages preview sandboxes JS events unreliably, but native CSS scroll-snap always works.

- **Print drop shadows need `print-color-adjust: exact`.** Chrome and Safari drop shadows from the print dialog unless this property is set. Apply to every card-class selector inside the `@media print` block.

- **Print page height = 209 mm (not 210).** The "-1 mm trick" prevents subpixel overflow that otherwise emits a blank trailing page in Chrome. The `skeletons.md` print skeleton has this baked in.

- **Brand mark is wordmark only, no decorative dot.** Earlier iterations had a teal circle alongside; that has been removed. The wordmark alone identifies the brand. The teal accent bar under every action title is the recurring brand visual; it does not need to be doubled.

- **Don't run sed-style replacement from inside a sandbox that may not let you delete `.bak` files.** Use `sed -i ''` on macOS or run cleanup via the file tools. Backup files surviving in the deliverable folder confuse Dropbox.

## What success looks like

The user opens the webview file in Safari on their laptop and it scales the 1280×720 frame to fit. They press `f` for fullscreen, `o` for overview, `←`/`→` for nav. They open the same file on iPhone — content reflows to single-column and scroll-snap takes over. They open it on iPad — same scroll-snap, content reflowed. They open the print file in Chrome, hit ⌘P, set Margins → None and Background graphics → on, and save a clean PDF.

If any of those four scenarios fail, something in the pipeline is off — go back and check the verification step.
