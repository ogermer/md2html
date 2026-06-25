# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repository is

This is **not** an application with a build/test pipeline. It is a single Claude Code **Skill** named `create-html-from-md`, packaged as documentation that Claude reads at runtime. The "code" is the Markdown instruction set plus the bundled HTML/CSS/JS snippets that Claude assembles into output files on demand. There is no install step, no dependency tree, and no test runner — the skill executes entirely by being read and followed.

The skill converts a Markdown deck source (numbered `# Slide N · Title` blocks with named content patterns) into two standalone HTML renderings carrying a **configurable brand visual system** (defined entirely in `brand.md`):
- **Web Viewer** (`<base>.html`) — on-screen, keyboard/touch nav, scroll-snap on phone & iPad.
- **Print** (`<base>-print.html`) — A4 landscape, one slide per page, tuned for the browser print dialog.

## Layout and the dependency chain

```
create-html-from-md/
├── SKILL.md      ← entry point: the step-by-step workflow Claude follows
├── brand.md      ← the single rebranding seam (palette, type, voice, file naming, layout constants)
├── assets/       ← canonical logo SVGs (blue for light bg, white for dark bg)
└── references/
    ├── skeletons.md     ← drop-in full-file HTML skeletons for both formats
    ├── patterns.md      ← per-pattern HTML snippets (§0 logo … §17 confidentiality footer)
    ├── webview-spec.md  ← canonical Web Viewer spec snapshot (v1.3)
    └── print-spec.md    ← canonical Print spec snapshot (v1.1)
```

The files form a deliberate hierarchy — understand it before editing any one of them:

- **`SKILL.md` is the orchestrator.** It defines the 7-step workflow and a Markdown-marker → HTML-pattern lookup table. It points to the other files rather than duplicating them.
- **`brand.md` is the rebranding seam.** Palette, typography, logo metadata, file-naming rule, and layout constants live here and *only* here. Changing a value here is intended to change every generated file. When asked to rebrand, edit `brand.md` (and swap `assets/` SVGs) — do not hardcode brand values into the skeletons.
- **`skeletons.md` + `patterns.md` are the assembly source.** Skeletons are whole-file templates; patterns are the per-slide blocks pasted into the skeleton's slide loop. These embed the literal CSS values from `brand.md`.
- **`*-spec.md` are intentional snapshots** of an upstream brand-guide, copied in so the skill is self-contained and works even if the upstream moves. If upstream changes, refresh these snapshots here directly — they are reference, not the assembly source.

## Working in this repo

There is nothing to build or lint. Editing the skill means editing Markdown. The only executable checks are the **output-verification commands** the skill runs *after generating a deck's HTML* (these validate generated files, not this repo). From `references/skeletons.md` § Verification commands and `SKILL.md` § Step 5:

```bash
# slide counts must balance across source + both outputs
grep -c '# Slide ' <base>.md
grep -c '<section class="slide" data-kind="content"' <base>.html
grep -c '<section class="page"' <base>-print.html   # = source count + 1 cover + N part dividers

# webview JS syntax — extract the <script> block and parse it
awk '/<script>/{flag=1;next} /<\/script>/{flag=0} flag' <base>.html > /tmp/_check.js
node --check /tmp/_check.js

# the webview's openPrint()/window.open must reference the real -print.html name
grep 'window.open' <base>.html
```

When generating a deck, follow `SKILL.md` exactly and in order — especially **Step 1** (always ask which format(s) to build) and **Step 1b** (confidentiality footer decision). Output files are written next to the Markdown source, never to an `outputs/` directory.

## Non-obvious invariants (these break decks every time)

These are documented in `SKILL.md` § Pitfalls and `brand.md`; the highest-leverage ones:

- **Logo CSS specificity.** Write `svg.brand-svg.blue { display: block }` / `svg.brand-svg.white { display: none }`. A `.slide-head .brand svg.brand-svg { display: block }` rule out-specifies the `.white` hide rule and renders both logos side-by-side.
- **`text-anchor="end"` on the wordmark.** The Inkscape SVG has whitespace on the right inside its viewBox; without right-edge anchoring the wordmark sits short of the slide-head's right padding.
- **Print page height is `209mm`, not 210** (the "-1 mm trick") — prevents a blank trailing page in Chrome. Non-negotiable.
- **Print drop shadows need `print-color-adjust: exact`** on every card-class selector inside `@media print`, or Chrome/Safari drop them in the print dialog.
- **iPad uses CSS scroll-snap, not JS handlers.** The stack-mode media query must be `(max-width: 640px), (any-pointer: coarse) and (max-width: 1400px)` — iPad preview sandboxes JS events unreliably.
- **Both logo SVGs are embedded once** as `<symbol>` defs in a hidden `<svg>` at the top of `<body>`, referenced via `<use href="#company-logo-blue">` / `…-white`. `assets/` holds the canonical source (placeholder `COMPANY_LOGO` wordmarks until branded).
- **Three-colour rule:** no more than three colour roles on any one slide. Navy 700 leads, Teal 500 accents, Amber is reserved for compliance/deadlines.
