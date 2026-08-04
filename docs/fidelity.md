# Fidelity

Phase 0 is an honest **static** renderer. This page summarises what works and
what does not; the authoritative, per-page assessment (three live pages with
committed golden PNGs) lives in the engine's
[`FIDELITY.md`](https://github.com/go-webengine/engine/blob/main/FIDELITY.md).

## Works today

- HTML parse → DOM → block/inline flow at a real viewport width.
- UA default styling for common tags (headings sized + bold, `<p>` margins, link
  colour, `<pre>` monospace, list/blockquote indents).
- Author CSS from `<style>` and inline `style=`, with cascade, specificity
  (inline > id > class > tag) and inheritance (`color`, `font-size`, weight,
  family, `text-align`, `white-space`).
- Colours: named, `#rgb`, `#rrggbb`, `rgb()`/`rgba()`; `background-color` boxes;
  the page backdrop (body/html background) extended over the whole viewport.
- Anti-aliased proportional text at the cascaded size, with correct advances
  driving greedy word-wrap; serif / sans / mono; faux-bold for weight ≥ 600;
  complex scripts (Cyrillic, Vietnamese, …) shaped legibly.
- `white-space: pre` (fixed-column `<pre>`).
- `<img>` best-effort: http(s) and `data:` sources, PNG/JPEG decode, downscale to
  viewport width.

## Not supported in Phase 0 (by design)

- **No JavaScript** — SPA / script-rendered content is blank or skeletal.
- **No float / flex / grid / table layout, no absolute/fixed positioning** —
  every page linearises to a single column; multi-column chrome stacks
  vertically.
- No `max-width`/`min-width`, `line-height`, `border`, `box-shadow`,
  `letter-spacing`, background images/gradients, `@media`, web fonts.
- Selectors: tag/class/id compound only — no combinators, attribute or pseudo
  selectors.
- Italic is not rendered (no italic face); `<em>`/`<i>` render upright.
- Margin collapsing is not implemented; inline whitespace at element boundaries
  is approximated.

## How to read a render

A modern CMS page (e.g. Wikipedia) renders **all its text content, legibly**, but
its float/flex/grid- and JS-driven visual chrome linearises — that is the
expected shape of a *static* render, not a bug. A plain-HTML page served as one
big `<pre>` (e.g. an RFC) preserves its monospace column layout faithfully. The
gap between the two is exactly the Phase 1–2 [roadmap](roadmap.md).
