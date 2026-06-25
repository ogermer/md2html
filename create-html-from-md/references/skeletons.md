# HTML skeletons — assembly guide

Two skeletons live in this skill: **Webview** and **Print**. They share the brand identity from `brand.md`, the pattern snippets from `patterns.md`, and the inline logo from `assets/`.

The full CSS for each format lives in the bundled spec files — don't restate hundreds of lines of CSS here. This file gives you the assembly order and the structural HTML; the CSS chunks live in:

- **Webview CSS:** `references/webview-spec.md` §7.1 (page/frame) → §7.7 (HUD), §14 (responsive). Copy each into the `<style>` block in order.
- **Print CSS:** `references/print-spec.md` §12 (minimum viable) + §6.5 (box weight & elevation) + §11 (print-dialog briefing CSS for the on-screen banner). Concatenate into the `<style>` block.

What's NOT in the spec files but IS required for v1.2+ output:

- The right-anchored SVG `<symbol>` block — see `patterns.md` §0.
- The CSS rule pair that picks blue vs. white logo by slide kind (below).
- The iPad-robust touch / keyboard handler set — see `webview-spec.md` §14.6.

---

## Webview skeleton

### Order of assembly

1. `<!DOCTYPE html>` + `<html lang="en">` + `<head>` with viewport meta, theme-color, Google Fonts pre-link.
2. `<style>` block containing, in order:
    1. `:root` palette + typography from `brand.md`.
    2. The entire webview-spec §7 CSS.
    3. The webview-spec §14 responsive media queries (`@media (max-width: 1024px)` + `@media (max-width: 640px), (any-pointer: coarse) and (max-width: 1400px)`).
    4. The v1.2 border-bump (× 1.3) — already baked into `brand.md` constants; apply when writing every border declaration.
    5. The v1.2 drop-shadow rule on card classes (see Drop-shadow CSS block below).
    6. The brand-mark CSS that picks blue vs white logo (see Logo display CSS block below).
3. `</style></head><body>`
4. The SVG logo `<symbol>` block from `patterns.md` §0.
5. The kbd-trap input: `<input class="kbd-trap" id="kbd-trap" aria-hidden="true" tabindex="-1" readonly inputmode="none" autocapitalize="off" autocomplete="off" autocorrect="off" spellcheck="false">`
6. The progress bar div: `<div class="progress"><div class="bar" id="progress-bar" style="width:0"></div></div>`
7. `<div class="stage" id="stage"><div class="frame" id="frame">`
8. **Slides** — for each slide in the markdown source, drop in the appropriate snippet from `patterns.md` (cover → §1, part dividers → §2, content slides → §3 + one of §4–§16).
9. `</div></div>` (close frame, close stage)
10. HUD: bottom-left current title + position, bottom-right key hints. See webview-spec §6.
11. Touch HUD pill (`.touch-hud`) with prev / position / next / overview / fullscreen buttons.
12. Overview grid container.
13. `<script>` block containing the full IIFE from webview-spec §8 + the v1.3 iPad-robust extensions (touch + pointer dedupe, keydown on document+body+kbd-trap, scroll-snap observer for tablet/phone modes).
14. `</body></html>`

### Logo display CSS block

Drop this immediately after the `.slide-head .brand` block in the slide-head CSS:

```css
.slide-head .brand {
  display: inline-flex; align-items: center;
  color: var(--slate-800); letter-spacing: -0.005em;
  text-transform: none; font-weight: 600; font-size: 13px;
  gap: 0;
  height: 22px;
}
.slide-head .brand svg.brand-svg { height: 22px; width: auto; }
/* Specificity-matched display toggle — both selectors carry (0,0,3,1).
   Writing this as `.brand .blue { display: block }` would lose to the broader
   `.brand svg.brand-svg` rule (because of the extra element) and both logos
   would render side-by-side. */
.slide-head .brand svg.brand-svg.blue  { display: block; }
.slide-head .brand svg.brand-svg.white { display: none; }
.slide.cover .slide-head .brand svg.brand-svg.blue,
.slide.part-divider .slide-head .brand svg.brand-svg.blue  { display: none; }
.slide.cover .slide-head .brand svg.brand-svg.white,
.slide.part-divider .slide-head .brand svg.brand-svg.white { display: block; }
```

### Drop-shadow CSS block

Apply to every card-class element. Place after §7.5 (content patterns) in the webview CSS:

```css
.stat-card, .window-card, .compare-card, .pillar, .arch-stage, .layer,
.flywheel-step, .buyer-card, .discovery-card, .level-step, .req-card,
.lane-row, .date-card, .mapping, .band-card {
  box-shadow: 0 1px 2.5px rgba(15, 20, 34, 0.08),
              0 0.5px 1px rgba(15, 20, 34, 0.06);
}
```

Do NOT apply to `.bottom-callout`, `.compare-callout`, `.cycle-callout` — callouts are flat by design.


### Confidentiality footer CSS

If the deck carries the optional confidential strap (see `SKILL.md` § Step 1b), every footer becomes a three-column grid so the strap sits centred between the existing left and right labels. Place after the slide-foot block in the webview CSS:

```css
/* v1.3+: optional CONFIDENTIAL — [NAME] [VERSION] strap centred in every footer */
.slide-foot { display: grid !important; grid-template-columns: 1fr auto 1fr; align-items: center; gap: 14px; }
.slide-foot > span:first-child  { text-align: left; }
.slide-foot > span.confidential { text-align: center; color: var(--danger); font-weight: 600; letter-spacing: 0.14em; }
.slide-foot > span:last-child   { text-align: right; }
/* On dark cover / part-divider slides, switch to amber so the strap reads as a watermark, not body text */
.slide.cover .slide-foot > span.confidential,
.slide.part-divider .slide-foot > span.confidential { color: var(--amber-500); }
```

Then in every footer (cover, part dividers, content slides), insert a third `<span class="confidential">CONFIDENTIAL — {NAME} {VERSION}</span>` between the existing left and right spans:

```html
<footer class="slide-foot">
  <span>{{LEFT_LABEL}}</span>
  <span class="confidential">CONFIDENTIAL — {{NAME}} {{VERSION}}</span>
  <span>{{RIGHT_LABEL}}</span>
</footer>
```

When the strap is not enabled, omit the middle span and the `display: grid` rule — the footer reverts to its original two-column `space-between` flex layout from the base spec.

### Responsive media query trigger (the iPad fix)

The stack-mode trigger needs to be exactly this:

```css
@media (max-width: 640px),
       (any-pointer: coarse) and (max-width: 1400px) {
  /* phone / tablet stack mode — content reflows, scroll-snap takes over */
  ...
}
```

The JS matchMedia:

```js
const phoneMq = window.matchMedia(
  '(max-width: 640px), (any-pointer: coarse) and (max-width: 1400px)'
);
```

This is the single most important v1.3 change. iPad fails JS-only navigation in QuickLook / Messages preview / Files-app preview; native CSS scroll-snap bypasses JS entirely and always works.

### openPrint() filename

The JS includes an `openPrint()` function bound to the `P` key. It must target the actual `-print.html` companion filename. Easy to forget after a version bump:

```js
function openPrint() {
  window.open('{{BASE_NAME}}-print.html', '_blank');
}
```

`{{BASE_NAME}}` matches the markdown source's basename (e.g. `example-deck-v1.2`).

---

## Print skeleton

### Order of assembly

1. `<!DOCTYPE html>` + `<html lang="en">` + `<head>` with viewport meta + Google Fonts pre-link.
2. `<style>` block containing, in order:
    1. `@page { size: A4 landscape; margin: 0 }`
    2. `:root` palette + typography + `--page-w: 297mm; --page-h: 209mm; --pad-x: 14mm; --pad-t: 11mm; --pad-b: 10mm` (the -1 mm trick — non-negotiable).
    3. The print-spec §12 minimum-viable CSS (`.page`, `.page-head`, `.page-foot`, `.page-body`, `.cols`).
    4. All pattern CSS adapted for mm units — see print-spec §5 for the catalogue, §7 for header/footer, §8 for type scale.
    5. The print-bleed rules from §6 (5 mm bleed on cover + part dividers).
    6. The v1.1 box-weight bump (border widths × 1.3 — apply when writing every declaration) — see print-spec §6.5.1.
    7. The v1.1 drop-shadow rule (see below).
    8. The `@media print` block with the `print-color-adjust: exact` rule for every card-class selector.
3. `</style></head><body>`
4. The print banner — visible only on screen, hidden in print. Loud, clear, mandatory:

   ```html
   <div class="print-banner">
     <strong>To print:</strong> open in <b>Chrome</b> · ⌘P · Destination: <b>Save as PDF</b> · Paper: <b>A4</b> · Layout: <b>Landscape</b> · Margins: <b>None</b> (critical) · Background graphics: <b>On</b> (critical) · Scale: <b>100%</b> · Headers and footers: <b>Off</b>.
   </div>
   <style>@media print { .print-banner { display: none !important; } }</style>
   ```

5. The SVG logo `<symbol>` block from `patterns.md` §0.
6. **Pages** — for each slide in the markdown source, emit a `<section class="page">` (or `.page.cover` / `.page.part-divider` for the cover and dividers). Use the same `patterns.md` snippets, swapping `.slide` → `.page`, `.slide-head` → `.page-head`, `.slide-foot` → `.page-foot`, `.slide-body` → `.page-body`.
7. `</body></html>`

### Print drop-shadow CSS block

```css
.stat-card, .window-card, .compare-card, .pillar, .arch-stage, .layer,
.flywheel-step, .buyer-card, .discovery-card, .level-step, .req-card,
.lane-row, .date-card, .mapping, .band-card {
  box-shadow: 0 0.25mm 0.7mm rgba(15, 20, 34, 0.08),
              0 0.1mm 0.25mm rgba(15, 20, 34, 0.06);
}
@media print {
  .stat-card, .window-card, .compare-card, .pillar, .arch-stage, .layer,
  .flywheel-step, .buyer-card, .discovery-card, .level-step, .req-card,
  .lane-row, .date-card, .mapping, .band-card {
    -webkit-print-color-adjust: exact !important;
    print-color-adjust: exact !important;
  }
}
```


### Confidentiality footer CSS (print)

Same three-column grid pattern as the webview, in mm units. Place after the print drop-shadow block:

```css
/* v1.3+: optional CONFIDENTIAL — [NAME] [VERSION] strap centred in every footer */
.page-foot { display: grid !important; grid-template-columns: 1fr auto 1fr; align-items: center; gap: 4mm; }
.page-foot > span:first-child  { text-align: left; }
.page-foot > span.confidential { text-align: center; color: var(--danger); font-weight: 600; letter-spacing: 0.14em; }
.page-foot > span:last-child   { text-align: right; }
.page.cover .page-foot > span.confidential,
.page.part-divider .page-foot > span.confidential { color: var(--amber-500); }
```

Footer HTML (same shape as webview, swap `slide-foot` → `page-foot`):

```html
<footer class="page-foot">
  <span>{{LEFT_LABEL}}</span>
  <span class="confidential">CONFIDENTIAL — {{NAME}} {{VERSION}}</span>
  <span>{{RIGHT_LABEL}}</span>
</footer>
```

The strap colour (`var(--danger)`) survives the print dialog because the existing `print-color-adjust: exact` block in the print spec already covers all card-class elements; the `.confidential` span doesn't need a separate declaration as long as the print banner reminds the user to enable Background graphics.

### Methodology table scoped overrides

If the markdown uses the methodology table (slide 11 pattern), include this scoped override in the print CSS so body cells fit. The `.page` selector beats `.page table td` (1,2 vs 1,1) — required for the override to apply.

```css
.page .method-table { font-size: 8.5pt; }
.page .method-table th { font-size: 7pt; }
.page .method-table td { font-size: 7.5pt; padding: 1.4mm 3mm; }
.page .method-table td:first-child { font-size: 9pt; color: var(--navy-700); font-weight: 600; }
.page .method-table td:first-child .src { font-size: 7.5pt; font-style: italic; color: var(--slate-400); }
```

### Deferral-card paragraph width override

If the markdown uses the big-date + deferral-card pattern (slide 15), include this scoped override so the deferral paragraphs use the full column width:

```css
.deferral-card p { max-width: none; margin: 0 0 1.5mm; }
.deferral-card p:last-child { margin-bottom: 0; }
```

Without this, the global `.page p { max-width: 145mm }` caps the deferral text early and the bottom callout overflows the page.

---

## Verification commands

After writing both files, run these checks (the SKILL.md step 5 has the full list):

```bash
# slide counts match
grep -c '<section class="slide" data-kind="content"' example-deck-v1.2.html
grep -c '# Slide ' example-deck-v1.2.md
grep -c '<section class="page"' example-deck-print-v1.2.html
# expect the source count + 1 cover + N part dividers in print

# webview JS syntax
awk '/<script>/{flag=1;next} /<\/script>/{flag=0} flag' example-deck-v1.2.html > /tmp/_check.js
node --check /tmp/_check.js && echo "JS OK"

# stale-template stragglers
grep -in 'previous-template-name\|previous-date' example-deck-v1.2.html  # expect zero

# placeholder tokens left unreplaced (expect zero in a finished, branded deck)
grep -no '{{[A-Z_]*}}\|COMPANY_LOGO' example-deck-v1.2.html

# print companion link in webview
grep 'window.open' example-deck-v1.2.html  # must reference the actual -print.html name

# both logo display rules survived
grep 'svg.brand-svg.blue\|svg.brand-svg.white' example-deck-v1.2.html | head -4
```
