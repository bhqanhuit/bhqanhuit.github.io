# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A personal academic portfolio site for Bui Huynh Quoc Anh, authored as **Claude Design "dc" documents** (`*.dc.html`) rather than a conventional web app. There is no package manager, build step, test suite, or git repo — the deliverable *is* the HTML files.

- `Academic Portfolio.dc.html` — the main page (real content: bio, news, publications, skills). Full document with `<helmet>`, design tokens, theming.
- `Publication List.dc.html` — a standalone filterable publication list fragment (still all `[placeholder]` content, not wired into the portfolio).
- `support.js` — the dc runtime. **Generated — never edit.** Its header points at `dc-runtime/src/*.ts`, which is not part of this repo.
- `portrait-circle.png`, `uploads/*` — images referenced by the portfolio.

## Running / previewing

Serve over HTTP and open the `.dc.html` file directly (no index page):

```bash
python3 -m http.server 8000     # then open http://localhost:8000/Academic%20Portfolio.dc.html
```

Notes:
- Network access is required — the runtime loads React 18 UMD (and Babel, only for JSX `x-import`) from unpkg with SRI.
- The `.dc.html` extension is load-bearing: `rootNameForDocument` derives the component name from it.
- `file://` mostly works but the runtime's self-refetch of the page is skipped; prefer the local server.
- Debugging: a throwing `renderVals()` renders a red `.sc-logic-error` banner over the component instead of crashing; import failures log under `[dc-runtime]` in the console.

## Document anatomy

Each `.dc.html` has exactly two authored parts:

1. `<x-dc>…</x-dc>` — the **template**. Plain HTML with inline styles, plus dc control-flow tags. Replaced at boot by `<div id="dc-root">`.
2. `<script type="text/x-dc" data-dc-script data-props="…">` — the **logic**, a `class Component extends DCLogic` (alias `StreamableLogic`) evaluated via `new Function`; only `DCLogic`, `StreamableLogic`, and `React` are injected.

The template renders against `{ ...props, ...renderVals() }`. `state` + `setState(updaterOrObject)` behave like React class components.

The escaped JSON in `data-props` declares `$preview` (`{width,height}` for the design-tool canvas) and per-prop editor metadata (`editor: text|enum|boolean`, `default`, `tsType`, `section`), surfaced as `this.props`.

## The template language — the main constraint to internalize

`{{ … }}` is **not** JavaScript. The evaluator supports only: identifier paths (`p.links`, `a[b]`), string/number/`true`/`false`/`null` literals, `!`, `==`/`!=`/`===`/`!==`, and parentheses. **No function calls, no arithmetic, no ternaries, no template literals.** Anything unsupported silently resolves to `undefined`.

Therefore every derived value — labels, conditional styles, formatted counts, event handlers — is precomputed in `renderVals()` and handed to the template as a plain key. Established patterns in this repo:

- Handlers as values: `onClick="{{ toggleTheme }}"`, where `renderVals()` returns `toggleTheme: () => this.setState(…)`. Per-item handlers are attached during `.map()` (see `toggleBib` in the portfolio, `chip()` in the publication list).
- Style branching: build the full inline style string in JS and bind `style="{{ c.style }}"` (`Publication List.dc.html:87`).
- Labels: `pubsMoreLabel`, `countLabel`, `themeIcon` — all string-built in JS.

Control flow and directives:

- `<sc-for list="{{ items }}" as="p" hint-placeholder-count="5">` — `as` name is the loop binding; a non-array `list` renders nothing and warns.
- `<sc-if value="{{ flag }}" hint-placeholder-val="{{ false }}">` — no `else`; use two complementary flags (`p.img` / `p.noImg`).
- `hint-placeholder-*` / `hint-size` only drive skeleton placeholders while streaming; they never affect the final render.
- `style-hover="…"` (and any `style-<pseudo>`) compiles to a generated CSS rule — the only way to get `:hover`/`:focus` given the inline-style-everything convention.
- `<helmet>` wraps `<title>`, meta/OG tags, font `<link>`s, and the one global `<style>` block (tokens, keyframes, `@media print`).
- `sc-camel-` attribute prefix forces a camelCase prop; `<x-import from="…" component="…">` and `<dc-import name="…">` pull in external/other components (unused here so far).

## Styling conventions

- All layout is inline styles on the element; the only shared CSS lives in the portfolio's `<helmet>` block.
- Design tokens are CSS custom properties on `:root` with a `[data-theme="dark"]` override; the theme is applied by `<div data-theme="{{ theme }}">` at the top of the template and flipped by `toggleTheme` state.
- `Publication List.dc.html` writes fallbacks everywhere (`var(--ink,#1C201D)`) because it can render outside the portfolio document that defines those tokens. Keep that habit for any new standalone fragment.
- `class="no-print"` marks interactive chrome hidden when printing (the CV/PDF path); `@media print` in the helmet strips animations and forces exact colors.

## Content state

Real content lives in `renderVals()` data arrays in `Academic Portfolio.dc.html`: `newsAll` and `pubsAll` (title, `authorsPre`/`authorsPost` around the bolded own-name, `venueLine`, `links`, `summary`, `img`). Publication thumbnails point into `uploads/` and are passed through `encodeURI` because several filenames contain spaces.

Several `renderVals()` keys are leftovers from the original template with `[Bracketed]` placeholder content and no template consumer: `projects`, `projectsShort`, `timeline`, `pubs`/`pubsShort`, `projectYears`, plus `initials`, `showFigures`, `menuOpen`/`toggleMenu`. Treat `[Bracket]` text as unfilled; don't assume an array is rendered without grepping the template for its key.
