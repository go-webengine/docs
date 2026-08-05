# Fidelity

An honest account of what the renderer matches in a browser and what it does not.
The authoritative, per-page and per-phase assessment (five live pages, committed
golden PNGs, before/after deltas) lives in the engine's
[`FIDELITY.md`](https://github.com/go-webengine/engine/blob/main/FIDELITY.md), and
the measured-vs-Chrome numbers in
[`bench/REPORT.md`](https://github.com/go-webengine/engine/blob/main/bench/REPORT.md).

## Measured vs headless Chrome

Windowed SSIM (`1.0` = identical) over the common top-left region at 1024px width;
`speed×` = `chrome_ms / webengine_ms` (>1 = webengine faster). Timings include the
live network fetch, so they vary run to run.

| URL | SSIM | pixdiff % | speed× | note |
|-----|-----:|----------:|-------:|:-----|
| example.com/ | **0.954** | 1.5 | 34.8 | near-parity, ~35× faster |
| react.dev/ | 0.727 | 26.4 | 1.15 | SPA; gradients + React SVG atom render |
| go.dev/blog/ | 0.670 | 33.2 | 0.34 | dark-mode + SVG logos render; slower |
| pkg.go.dev/net/http | 0.629 | 36.5 | 0.14 | large computed page; perf gap |
| en.wikipedia.org/wiki/Go | 0.441 | 22.4 | 1.20 | JS-confounded, noisy metric |

Mean windowed-SSIM ≈ **0.69**. `example.com` is at near-parity and much faster;
the JS-heavy and large computed pages are the honest frontier. The Wikipedia
number is confounded by runtime JS chrome and swings run-to-run around 0.44.

## Works today

- **Full box-model layout**: block/inline flow, floats + clear, flexbox, CSS
  grid, tables, `position` (relative/absolute/fixed/sticky), margin collapsing,
  greedy word-wrap.
- **CSS**: cascade + specificity (inline > id > class > tag) + inheritance;
  `var()` custom properties; `@media` width queries; **dark-mode**
  (`prefers-color-scheme`); external `<link>` stylesheets; UA defaults.
- **Selectors**: tag/class/id/compound, descendant + child + **sibling (`~`/`+`)**
  combinators, **`:checked`**, **`:not()`** (the checkbox-hack that collapses
  MediaWiki dropdowns); unmodelled selectors reduce rather than drop the rule.
- **Colour & decoration**: named/`#rgb`/`#rrggbb`, modern `rgb()`/`hsl()`,
  `background-color`, **linear/radial gradients**, `background-image: url()`,
  border + **border-radius**, **box-shadow**, group **opacity**.
- **Text**: anti-aliased proportional text with **real bold and italic** faces
  (no faux-bold); serif / sans / mono; complex scripts (Cyrillic, Vietnamese, …);
  `white-space: pre`.
- **Images**: `<img>` over http(s) + `data:` (PNG/JPEG) and **SVG** (oksvg/rasterx)
  via `<img *.svg>`, `data:image/svg+xml` and inline `<svg>`.
- **JavaScript**: page scripts run via [goja](https://github.com/dop251/goja)
  against a real DOM, with `fetch()`/XHR and read-back of real laid-out geometry
  (`getBoundingClientRect`, `offset*`, `getComputedStyle`). A settle-then-render
  loop re-cascades and re-lays-out after scripts mutate the DOM (including
  dynamically injected `<script>`/`<style>`/`<link>`), to a bounded fixpoint. The
  same JS-settled DOM drives the click hit-map.

## Not supported yet (stated, not hidden)

- No `conic-gradient`, CSS `filter` or `mask`. SVG has no `<filter>`/`<mask>`/
  `<pattern>`/`<text>`/embedded `<image>`, and a per-page image budget caps very
  icon-heavy pages.
- No `<li>` `list-style` marker discs; some icon-font / `visually-hidden` chrome
  renders as text where a browser shows an icon.
- Large computed pages (pkg.go.dev, go.dev) render **slower** than Chrome — a perf
  gap, not a fidelity one.
- This is **not** a standards-complete browser and is **not** claimed to match
  Chromium pixel-for-pixel. It is an honest renderer whose gap to a browser is
  the [roadmap](roadmap.md), measured page by page rather than asserted.

## How to read a render

`example.com` renders at near-parity. A JS-driven SPA (react.dev) renders its
gradients, SVG atoms and script-built content and lands around 0.73 SSIM. A large
computed docs page (pkg.go.dev) renders correctly but slower than Chrome. A modern
CMS page (Wikipedia) collapses its Vector-2022 chrome via the checkbox-hack and
`mw.loader`, but the SSIM stays noisy where runtime chrome and icon fonts differ —
the residual is documented, not hidden. The gap between these is exactly the
remaining-levers section of the [roadmap](roadmap.md).
