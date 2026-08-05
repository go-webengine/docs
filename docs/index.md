# go-webengine

A pure-Go, **`CGO_ENABLED=0`** headless web engine. Give it a URL, get back an
**image of the page** — HTML/CSS layout and paint, with **JavaScript executed**
against a real DOM, and **no Chromium, no cgo and no host web view**.

It is the rendering core of the
[`browserproxy`](https://github.com/go-webengine/browserproxy) remote-browser
service and the [wasmdesk](https://github.com/wasmdesk) in-desktop browser: a
server renders pages and streams frames (plus a click hit-map) to a thin client.

## What it renders today

- **Full box-model layout**: block, inline, floats + clear, **flexbox**, **CSS
  grid**, **tables**, **`position`** (relative/absolute/fixed/sticky), margin
  collapsing, word-wrap.
- **Broad CSS**: cascade + specificity + inheritance, `var()` custom properties,
  `@media` width queries, **dark-mode**, external `<link>` stylesheets, modern
  `rgb()`/`hsl()`, **linear/radial gradients**, `background-image`, border +
  **border-radius**, **box-shadow**, opacity; sibling combinators, `:checked`,
  `:not()`.
- **Text**: anti-aliased serif/sans/mono with **real bold and italic** faces.
- **Images**: PNG/JPEG, `data:`, and **SVG** (`<img>`, `data:` and inline `<svg>`).
- **JavaScript**: page scripts run via [goja](https://github.com/dop251/goja)
  against a real DOM with `fetch()`/XHR and laid-out-geometry read-back; a
  settle-then-render loop reflects script-driven DOM mutations in the output.

See **[Fidelity](fidelity.md)** for the honest, measured account of what matches a
browser and what does not.

## Pipeline

```
Fetch (go-browserhttp)  →  Parse (x/net/html → dom)  →  Cascade (css: +var()/@media/dark-mode)
     →  JavaScript (js: goja + real DOM + fetch/XHR)
     →  Layout: block · inline · float · flex · grid · table · position (layout)
     ⟲   settle loop: JS reads laid-out metrics → re-cascade + re-layout to a bounded fixpoint
     →  Paint: AA text/SVG/gradients/shadows + backgrounds (go-widgets/painter) + images (go-images)
     →  image.RGBA  →  PNG (+ optional link hit-map)
```

## Quickstart

```go
import (
    "context"
    "image"
    "os"

    "github.com/go-webengine/engine"
)

png, err := engine.Screenshot(
    context.Background(),
    "https://example.com/",
    image.Rect(0, 0, 1024, 768),
)
if err != nil {
    panic(err)
}
_ = os.WriteFile("out.png", png, 0o644)
```

`engine.Render` returns the raw `*image.RGBA` plus a `RenderInfo` (`Title`,
`URL`, `ContentHeight`). For more control, `engine.New()` gives an `*Engine` with
`RenderHTML` (offline), `RenderWithLinks` (image + click hit-map) and a
`DisableJS` field. The viewport width is fixed; the height grows to fit the whole
page. A `render` CLI is bundled:

```
go run github.com/go-webengine/engine/cmd/render \
  -url https://example.com/ -out out.png -w 1024 -h 768
```

## Where to next

- **[Architecture](architecture.md)** — the packages, the reuse-vs-build decision,
  and why prior pure-Go browsers were studied but not built on.
- **[Roadmap](roadmap.md)** — the phases from static render to the shipping
  remote-browser proxy, and the remaining levers.
- **[Fidelity](fidelity.md)** — an honest, measured account of what renders like a
  browser today and what does not.

## Links

- Source: <https://github.com/go-webengine/engine>
- Remote-browser service: <https://github.com/go-webengine/browserproxy>
- Landing: <https://go-webengine.github.io/>
- License: BSD-3-Clause
