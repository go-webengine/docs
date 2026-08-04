# Roadmap

The engine grows in phases. Only **Phase 0 ships today**; the rest is the
intended direction, stated honestly rather than promised.

## Phase 0 — Static renderer · shipping

A static (no-JavaScript) renderer end to end:

- HTML parse → owned DOM.
- A real CSS cascade: UA default stylesheet, `<style>` and inline `style=`,
  tag/class/id selectors with specificity (inline > id > class > tag), and
  inheritance.
- Block-and-inline flow layout with a greedy word-wrap line-breaker and
  `white-space: pre`.
- Anti-aliased text (go-opentype) at the cascaded size, backgrounds, and
  best-effort `<img>` (http(s) + `data:`, PNG/JPEG), painted to an
  `image.RGBA` → PNG.

See [Fidelity](fidelity.md) for exactly what renders correctly and what does not.

## Phase 1 — Scripting · planned

Wire a pure-Go JavaScript engine ([goja](https://github.com/dop251/goja) — no
cgo, ES5.1 + most ES2015) to the DOM so script-rendered content populates.
Deferred deliberately: Phase 0 has no JS dependency at all.

## Phase 2 — Real box model · planned

`float` / `flex` / `grid` / table layout and absolute/fixed positioning, so
modern page chrome (sidebars, nav bars, infoboxes) no longer linearises to a
single column. Selector combinators, `max-width`/`min-width`, `line-height`,
`border` and more CSS follow here. The clean CSS-selector upgrade path
(cascadia / tdewolff) is already scoped in the survey.

## Phase 3 — browserproxy · planned

Render server-side and stream frames to the
[wasmdesk](https://github.com/wasmdesk) `clients/browser` front-end — the
original motivation for a headless, cgo-free engine that runs anywhere Go runs.

## Non-goals (for now)

- Being a full standards-compliant browser. This is a *renderer* with an honest,
  growing feature set, not a Chromium replacement.
- cgo, a bundled browser binary, or a host web view — ever. Pure Go is the whole
  point.
