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
| example.com/ | **0.831** | 5.6 | 18.3 | near-parity; its JS cross-fade is not modelled |
| en.wikipedia.org/wiki/Go | 0.417 | 30.3 | 1.6 | live article; JS and rate-limited thumbnails |
| pkg.go.dev/net/http | 0.716 | 11.3 | 0.8 | large computed page |
| go.dev/blog/ | 0.700 | 16.6 | 0.45 | slower than Chrome |
| react.dev/ | 0.725 | 33.0 | 1.3 | SPA; hydration fails, so the static fallback renders |
| news.ycombinator.com/ | 0.615 | 13.7 | 1.75 | table layout; rotating front page |
| developer.mozilla.org/…/CSS | 0.606 | 18.1 | 3.3 | docs layout |
| github.com/golang/go | 0.637 | 14.2 | 2.0 | live repository counters |
| tailwindcss.com/ | 0.724 | 12.2 | 0.5 | sponsor carousel rotates; slower |
| caniuse.com/ | 0.660 | 18.0 | 0.8 | data grid |

Mean SSIM over the ten pages is **≈ 0.66**. Summed over the run, webengine took
26.1 s and headless Chrome 26.3 s, so overall time is at parity, but the per-page
spread is wide. The Wikipedia number is confounded by runtime JS chrome and by
rate-limited image thumbnails.

## Works today

- **Full box-model layout**: block/inline flow, floats + clear, flexbox, CSS
  grid, tables (with `vertical-align` on cells), `position`
  (relative/absolute/fixed/sticky), margin collapsing, greedy word-wrap, multi-column
  layout.
- **Effects and lists**: `translate`/`rotate`, `filter`, `backdrop-filter`,
  `mask-image` (a single `url()` mask), `ul`/`ol` list markers.
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

- `conic-gradient` is recognised but not painted. `filter` and `mask-image` are
  modelled narrowly: a mask is a single `url()` stretched over the box. `scale`
  and `skew` are not supported. SVG has no `<filter>`/`<mask>`/`<pattern>`/
  `<text>`/embedded `<image>`, and a per-page image budget caps very icon-heavy
  pages.
- `::before`/`::after` generated content is not synthesised, so some icon-font and
  `visually-hidden` chrome renders as text where a browser shows an icon.
- Rate-limited image hosts (Wikimedia) answer some requests with HTTP 429. The
  engine retries them and the renders are identical, but each retry adds about two
  seconds to that page.
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
