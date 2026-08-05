# Roadmap

The engine grew in phases. **Phases 0 through 2.4 have shipped**, and the
[`browserproxy`](https://github.com/go-webengine/browserproxy) remote-browser
service that was the original Phase-3 goal is **live**. What remains is a set of
honest levers, not a promise of a full browser. Measured fidelity is on the
[Fidelity](fidelity.md) page; the phase-by-phase log with before/after numbers is
in the engine's
[`FIDELITY.md`](https://github.com/go-webengine/engine/blob/main/FIDELITY.md) and
[`bench/REPORT.md`](https://github.com/go-webengine/engine/blob/main/bench/REPORT.md).

## Shipped

| Phase | What landed |
|---|---|
| **0 — Static renderer** | HTML → owned DOM → real CSS cascade (specificity + inheritance) → block/inline flow → AA text/background/image paint → PNG. |
| **1 — Real box model** | Floats + clear, flexbox, tables, painted borders, margin collapsing, `@media` width queries. |
| **1.5 — CSS breadth I** | External `<link>` stylesheet fetch, `var()` custom properties, anchor **click hit-map** (`Links`). |
| **1.7 — Flex + grid** | Complete flexbox and **CSS grid**. |
| **1.8 — Positioning** | `position: relative / absolute / fixed / sticky`. |
| **1.9 — CSS breadth II** | Modern colour (`rgb()`/`hsl()`), Tailwind variants, **border-radius**, dark-mode groundwork. |
| **2.0 — Decorations** | **Linear/radial gradients**, `background-image: url()`, **box-shadow**, group **opacity**. |
| **2 — JavaScript** | [goja](https://github.com/dop251/goja) bound to a real DOM with `fetch()`/XHR. |
| **2.1 — SVG** | Pure-Go SVG rasterisation (oksvg + rasterx) for `<img *.svg>`, `data:` and inline `<svg>`. |
| **2.2 — Dynamic render** | Layout↔JS metric read-back (`getBoundingClientRect`, `offset*`, `getComputedStyle`) + a **settle-then-render** bounded-fixpoint loop; runtime-injected `<script>`/`<style>`/`<link>`. |
| **2.3 — Selector hack** | Sibling combinators `~`/`+`, **`:checked`**, **`:not()`** — the checkbox-hack that collapses MediaWiki dropdowns; "reduce, don't drop" for unmodelled selectors. |
| **2.4 — JS on hit-map** | The `RenderWithLinks` path shares the JS-settled pipeline, so JS-built links land in the click hit-map. |
| **Remote browser** | [`browserproxy`](https://github.com/go-webengine/browserproxy): a WebSocket service that renders server-side with the engine, streams frames + hit-map, forwards clicks/scrolls/keys, and guards against SSRF. **Shipping.** |

Real bold + italic faces replaced the earlier faux-bold synthesis along the way.

## Remaining levers (honest, not promised)

- **Fidelity residuals** — `<li>` `list-style` marker discs; icon-font /
  `visually-hidden` chrome that renders as text; `conic-gradient`, CSS `filter`
  and `mask`; SVG `<filter>`/`<mask>`/`<pattern>`/`<text>`/embedded `<image>`.
- **Performance** — large computed pages (pkg.go.dev, go.dev) render slower than
  Chrome; that is the next big lever, and it is a speed gap, not a fidelity one.
- **Interaction depth in JS** — more of the CSSOM and event model, so highly
  interactive SPAs settle closer to their browser state.

These are diminishing-returns work: the mean windowed-SSIM across the five bench
pages is already ≈ 0.69, and `example.com` is at near-parity (0.954).

## Non-goals (for now)

- Being a full standards-compliant browser. This is a *renderer* with an honest,
  growing feature set — **not** a Chromium replacement, and not claimed to match
  Chromium pixel-for-pixel.
- cgo, a bundled browser binary, or a host web view — ever. Pure Go is the whole
  point.
