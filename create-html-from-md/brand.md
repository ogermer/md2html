# Brand configuration — {{COMPANY_NAME}}

**Edit this file to brand the skill.** Everything below is read by `SKILL.md` step 2 and threaded through the bundled HTML skeletons (`references/skeletons.md`) and pattern snippets (`references/patterns.md`). Change a value here, regenerate, and the look-and-feel follows.

> **This is a clean, shareable template.** All organisation-specific identity has been replaced with `{{PLACEHOLDER}}` tokens and a neutral default palette. Before first use, work through the **Placeholders to replace** checklist at the bottom of this file and swap in the real brand. Until you do, generated decks render with `COMPANY_LOGO` placeholder wordmarks and `{{…}}` tokens shown verbatim.

---

## Brand identity

- **Name:** {{COMPANY_NAME}}
- **Positioning chip (cover):** `{{COMPANY_TAGLINE}}`
- **Default presenter:** {{PRESENTER_NAME}} · `{{PRESENTER_EMAIL}}`
- **Brand voice note:** Lead with the primary (navy) colour, accent with the secondary (teal) colour, reserve the alert (amber) colour for compliance / deadlines. Action titles, not topic titles. Every number carries an inline source. Keep a short list of banned phrases the brand never uses (a starting example: `transformative · AI-native · magical · seamless · revolutionise · just works · vendor-neutral · single source of truth · 10x · cutting-edge · empower · democratise · unleash · supercharge`). If the brand has a recurring verbal signature — a set phrase or cadence that should recur across decks — record it here as `{{BRAND_VOICE_SIGNATURE}}`.

## Palette (CSS custom properties)

These are the literal values written into the `:root { … }` block of every generated HTML file. Hex values without leading `#`. The defaults below are a neutral navy/teal/amber system — replace the hex values to match the brand.

```css
--navy-900:#06245A; --navy-700:#0B3B8C; --navy-500:#2E5BBF; --navy-50:#EAF1FB;
--teal-700:#16998A; --teal-500:#1FB8A0; --teal-50:#EAF7F4;
--amber-700:#C7770A; --amber-500:#F39C12; --amber-50:#FCEDB7;
--slate-950:#0F1422; --slate-800:#1F2A3D; --slate-700:#2C3A52; --slate-600:#4A5A70;
--slate-400:#8A95A8; --slate-200:#E2E6EC; --slate-100:#EEF1F5; --slate-50:#F8FAFC;
--white:#FFFFFF; --success:#28A745; --danger:#D9534F;
```

Compare-card tints (red/green):

```css
/* compare-card backgrounds */
without-bg:#FBEFEF; without-border:#F1D6D6;
with-bg:   #ECF7EF; with-border:   #CFE7D6;
```

Three-colour rule: no more than three colour roles on any one slide.

## Typography

```css
--sans:  'Inter', -apple-system, BlinkMacSystemFont, 'Segoe UI', sans-serif;
--serif: 'Source Serif 4', Georgia, 'Times New Roman', serif;
```

Inter for everything except long-form editorial passages (founder letters, manifestos). Source Serif 4 only where it earns its keep. Tabular figures (`font-feature-settings: "tnum" 1`) on stat cards, tables, dates, and any number-heavy region.

Google Fonts pre-link is always included in the HTML so the file works online; falls back gracefully offline to the platform sans-serif chain.

## Logos

```
assets/company-wordmark-blue.svg      # dark/navy stroke + fill — use on bright/slate-50 backgrounds (content slides)
assets/company-wordmark-white.svg     # light stroke + fill — use on navy/dark backgrounds (cover, part dividers)
```

Both files ship as **placeholders**: each is a right-anchored `<text>COMPANY_LOGO</text>` inside the native viewBox. Replace them with the real wordmark SVGs (keep the viewBox so the layout constants below stay valid), or paste the real wordmark text into the `<text>` element.

Native viewBox: `0 0 434.10657 95.619486` (aspect ~4.54:1). Rendered at `height: 22px` in webview, `height: 6mm` in print → natural width ≈ 100 px / 27 mm.

**The wordmark needs `text-anchor="end"` with x at the viewBox right edge** so the right edge of the wordmark aligns flush with the slide-head's right padding. The bundled `patterns.md` § Logo symbols snippet already does this. Do not skip — many exported wordmark SVGs carry whitespace on the right inside their viewBox that would otherwise leave a visible gap.

Wordmark only by default. If the brand has a separate icon/mark glyph, decide deliberately whether to pair it with the wordmark; the recurring teal accent bar under every action title already carries the brand visual, so doubling it in the header is usually noise.

## File naming

For a markdown source at `<path>/<base>.md`, the generated outputs are:

| Format | Path |
|---|---|
| Webview | `<path>/<base>.html` |
| Print | `<path>/<base>-print.html` |

If the source has a version suffix (e.g., `example-deck-v1.2.md`), it carries through to both outputs unchanged.

The webview's `openPrint()` JavaScript function must target the actual `-print.html` filename — easy to forget after a version bump.


## Confidentiality footer (optional)

If the deck is for a single named recipient (a one-investor pitch, a customer-specific proposal, an internal-only review), every slide can carry a centred confidentiality strap in the footer alongside the section label (left) and the slide number (right).

**Template:** `CONFIDENTIAL — [NAME] [VERSION]`

- `[NAME]` is the audience-facing name in upper case — usually drawn from the deck title (e.g. `CLIENT NAME`, `BOARD OF DIRECTORS`, `INTERNAL ONLY`).
- `[VERSION]` is the edition with the word `VERSION` in front (e.g. `VERSION 1.3`).

**Example:** `CONFIDENTIAL — CLIENT NAME VERSION 1.3`

**Defaults.**

- **Webview / print colour:** danger-red (`var(--danger)`) on light slate-50 content slides; amber (`var(--amber-500)`) on the dark navy cover and part dividers so it reads as a watermark, not body text.
- **Letter-spacing:** `0.14em` (webview) / equivalent in print. Bold weight. Slightly heavier than the surrounding footer labels so the strap is recognisable when flipping through the deck.
- **Layout:** the footer becomes a three-column grid (`1fr auto 1fr`) — section label / centred strap / slide number — so the existing left and right elements keep their positions.

**When to include it.**

| Deck type | Confidentiality footer |
|---|---|
| Single-recipient pitch | Yes |
| Customer / partner proposal | Yes |
| Internal-only review | Yes (with `INTERNAL ONLY` as `[NAME]`) |
| Public marketing collateral | No |
| Conference talk slides | No |

If the deck has no specific audience, omit the strap entirely — the footer reverts to its two-column form.

The skill's workflow (`SKILL.md` § Step 1) asks whether to include this footer when it isn't obvious from the markdown source.

## Current edition metadata (the values to drop into the cover meta-row by default)

The user usually overrides these per-deck. Defaults here are what to suggest if the markdown source's leading metadata block is missing.

- **Edition:** *(read from the markdown source's `**Edition:**` line; otherwise prompt)*
- **Date:** *(read from `**Date:**` line; otherwise today)*
- **Presenter:** {{PRESENTER_NAME}} · `{{PRESENTER_EMAIL}}`
- **Format:** 30-minute working session
- **Location:** (only if the source mentions a specific meeting; otherwise omit the column and use `FORMAT` instead)

## Layout constants — webview

```
slide-w: 1280px
slide-h: 720px
slide padding: 36px 64px 32px
slide-head font-size: 10.5px
slide-foot font-size: 9.5px
brand mark height: 22px
accent bar: 48px × 2.5px, teal-500
border weight: 1.3 × the original CSS-baseline (hairline 1.3px, accent 3.9–5.2px) — see references/skeletons.md
card drop shadow: 0 1px 2.5px rgba(15,20,34,0.08), 0 0.5px 1px rgba(15,20,34,0.06)
```

## Layout constants — print

```
page size: A4 landscape, 297 × 210 mm
page-h CSS value: 209mm  (the -1mm trick — non-negotiable)
page padding: 11mm 14mm 10mm
page-head font-size: 8pt
page-foot font-size: 8pt
brand mark height: 6mm
accent bar: 12mm × 1mm, teal-500
border weight: 1.3 × the original mm baseline (hairline 0.26mm, accent 0.78–1.43mm)
card drop shadow: 0 0.25mm 0.7mm rgba(15,20,34,0.08), 0 0.1mm 0.25mm rgba(15,20,34,0.06)
print-color-adjust: exact   (so the shadow + bg colour survive the print dialog)
```

## Placeholders to replace

This template ships with neutral tokens. Search the whole skill folder for each token below and replace every occurrence before generating real decks. (`grep -rn '{{' .` lists them all.)

| Token | What it is | Example replacement |
|---|---|---|
| `{{COMPANY_NAME}}` | The brand / organisation name | `Example Inc.` |
| `{{COMPANY_TAGLINE}}` | Cover positioning chip — a short capitalised strap | `THE OPERATING SYSTEM FOR FIELD TEAMS` |
| `{{PRESENTER_NAME}}` | Default deck presenter / owner | `Jane Doe` |
| `{{PRESENTER_EMAIL}}` | Default presenter email | `jane@example.com` |
| `{{BRAND_VOICE_SIGNATURE}}` | Optional recurring verbal signature / cadence | *(brand-specific phrase)* |
| `COMPANY_LOGO` (in `assets/*.svg`) | Placeholder wordmark text | Replace the SVG with the real wordmark |

Symbol IDs in the embedded logo block use the neutral names `company-logo-blue` / `company-logo-white` — no need to change them, but they can be renamed if you prefer.

## What to update when rebranding

1. Name + positioning chip + presenter (top of this file) — the `{{…}}` tokens above.
2. Palette block (CSS custom properties).
3. Typography (`--sans`, `--serif`, Google Fonts URL in the skeletons).
4. Logos in `assets/` — replace both SVG files; update the `viewBox` if the new logo has a different aspect.
5. The voice rules — paste in the new brand's house-style notes and `{{BRAND_VOICE_SIGNATURE}}`.
6. Run a test build on a sample deck and visually QA at least the cover, a content slide with the three-windows pattern, and the print version.

The layout constants and the responsive breakpoints (640 px / 1400 px / `any-pointer: coarse`) are brand-agnostic — don't touch unless you have a specific reason.
