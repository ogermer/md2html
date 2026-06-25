# md2html

A [Claude Code](https://claude.com/claude-code) **skill** that converts a Markdown deck source into two standalone, on-brand HTML renderings:

- **Web Viewer** (`<base>.html`) — on-screen, keyboard/touch navigation, scroll-snap on phone & iPad.
- **Print** (`<base>-print.html`) — A4 landscape, one slide per page, tuned for the browser print dialog.

Both outputs are fully self-contained (no runtime network dependencies; fonts pre-linked but degrade gracefully offline), and the logo is embedded inline as an SVG `<symbol>`.

## This is a clean, brand-agnostic template

All organisation-specific identity has been replaced with `{{PLACEHOLDER}}` tokens and a neutral default palette, so the skill can be dropped into any corporate environment and rebranded. The logos ship as placeholder wordmarks reading `COMPANY_LOGO`.

**Before first use**, work through the **Placeholders to replace** checklist at the bottom of [`create-html-from-md/brand.md`](create-html-from-md/brand.md) — that file is the single rebranding seam (palette, typography, logos, voice, file naming). Find every token with:

```sh
grep -rn '{{' create-html-from-md
grep -rn 'COMPANY_LOGO' create-html-from-md
```

## Layout

```
create-html-from-md/        ← the skill
├── SKILL.md                ← the step-by-step workflow Claude follows
├── brand.md                ← rebranding seam: palette / type / logos / voice / naming + placeholder checklist
├── assets/                 ← placeholder logo SVGs (COMPANY_LOGO); replace when branding
└── references/
    ├── skeletons.md         ← drop-in full-file HTML skeletons (webview + print)
    ├── patterns.md          ← per-pattern HTML snippets (cover, three-stat, three-windows, …)
    ├── webview-spec.md      ← canonical Web Viewer spec
    └── print-spec.md        ← canonical Print spec
example/                    ← a rendered sample deck (source + both HTML views + PDF)
CLAUDE.md                   ← architecture notes for Claude Code
```

## Install

Copy the `create-html-from-md/` directory into your Claude Code skills location (e.g. a plugin's `skills/` directory or `~/.claude/skills/`), then ask Claude to "build the webview/print HTML from `<your-deck>.md`".

## Example

The [`example/`](example/) folder contains a sample deck (`example-deck-v1.0.md`) and its generated Web Viewer, Print HTML, and a PDF exported from the print edition — built from this template with placeholders left in place, so you can see exactly what the skill produces.
