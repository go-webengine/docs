# Architecture

The engine is a linear pipeline from bytes on the wire to pixels in an image.
Each stage is an owned package with a narrow contract, so the pure-logic stages
(cascade, layout, DOM) stay testable without a network or fonts.

## Packages

| Package | Role |
|---|---|
| `dom` | An owned DOM node tree built from [`golang.org/x/net/html`](https://pkg.go.dev/golang.org/x/net/html) — a thin, owned wrapper so the rest of the engine never depends on `html.Node` directly. |
| `css` | A minimal-but-real CSS subset: value model, stylesheet/declaration parser, tag/class/id selectors with specificity, a UA default stylesheet, and cascade + inheritance. Its own small tokenizer/parser, for full control and 100%-coverable pure logic. |
| `layout` | Block-and-inline flow with a greedy word-wrap line-breaker and `white-space: pre`, driven through a `Measurer` interface so geometry is unit-testable **without fonts**. |
| `paint` | Rasterises the box tree to `*image.RGBA`; also provides the real `Measurer` backed by go-opentype faces. |
| `engine` (root) | The public API — `Fetch`, `Render`, `Screenshot`, `RenderInfo` — plus image sub-resource loading. |
| `cmd/render` | A single-binary CLI: `render -url URL -out shot.png -w 1024 -h 768`. |

## The `Measurer` seam

Layout never touches a font directly. It asks a `Measurer` for advance widths and
line metrics. In production the measurer is backed by go-opentype faces; in tests
it is a deterministic fake, so box geometry can be asserted at **exact pixel
positions** with no font files and no anti-aliasing noise. This is why the layout
and cascade packages carry very high statement coverage.

## Reuse vs build

Everything reused is pure-Go and permissively licensed (BSD-2/BSD-3/MIT), so the
whole engine builds with `CGO_ENABLED=0` and carries no copyleft.

**Reused**

| Module | Role |
|---|---|
| [`go-browserhttp/browserhttp`](https://github.com/go-browserhttp/browserhttp) | `http.Client` with a Chrome TLS fingerprint, cookie jar and redirect following. |
| [`golang.org/x/net/html`](https://pkg.go.dev/golang.org/x/net/html) | HTML5 tokenizer + tree builder. |
| [`go-opentype/opentype`](https://github.com/go-opentype/opentype) + [`fonts`](https://github.com/go-opentype/fonts) | Advance-width measurement and 8-bit AA glyph masks; embedded OFL sans/serif/mono faces. |
| [`go-widgets/painter`](https://github.com/go-widgets/painter) | `FillRect` / clip for backgrounds onto the RGBA buffer. |
| [`go-images/images`](https://github.com/go-images/images) | PNG/JPEG decode + resize for `<img>`. |

**Built clean-room** — the DOM wrapper, the CSS cascade + inheritance, the
block/inline flow layout engine, and the paint orchestration + public API.

## Why not build on a prior pure-Go browser?

The prior pure-Go browser engine `opossum` (renamed to
[`mycel`](https://github.com/psilva261/mycel)) was studied in depth. Its choice
of permissive parse/selector/tokenize building blocks is worth emulating, but its
**layout core is explicitly stub-quality** (its README calls float/flex layout
"just stub implementations"), and its rendering backend is a Plan 9 pixel toolkit
(`duit`) with the DOM exposed as a 9P filesystem — both dead weight for a
*headless* engine. The single hardest piece of a browser, layout, is exactly what
it cannot give us, so the layout/box/flow engine here is clean-room. The full
prior-art verdict lives in the engine's
[`SURVEY.md`](https://github.com/go-webengine/engine/blob/main/SURVEY.md).

## Portability & testing

Pure Go with `CGO_ENABLED=0`; cross-built in CI for all six of Go's 64-bit
targets (amd64, arm64, riscv64, loong64, ppc64le, s390x). A ratchet coverage gate
holds the pure-logic packages at their measured floor and raises it toward 100%
as the engine matures; a committed golden PNG covers the offline paint path. The
live-network paths are excluded from the gate because their coverage is not
reproducible in CI.
