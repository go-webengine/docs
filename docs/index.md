# go-webengine

A pure-Go, **`CGO_ENABLED=0`** headless web engine. Give it a URL, get back an
**image of the page** — HTML/CSS layout and paint with **no Chromium, no cgo and
no host web view**.

It is the rendering core of the [wasmdesk](https://github.com/wasmdesk)
**browserproxy** roadmap: a service that renders pages server-side and streams
frames to a thin front-end. Phase 0 — shipping today — is a *static* renderer
(no JavaScript).

## Pipeline

```
Fetch (go-browserhttp)  →  Parse (x/net/html → dom)  →  Cascade (css)
     →  Layout: block + inline flow, word-wrap (layout)
     →  Paint: AA text (go-opentype) + backgrounds (go-widgets/painter) + images (go-images)
     →  image.RGBA  →  PNG
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
`URL`, `ContentHeight`). The viewport width is fixed; the height grows to fit the
whole page. A `render` CLI is bundled:

```
go run github.com/go-webengine/engine/cmd/render \
  -url https://example.com/ -out out.png -w 1024 -h 768
```

## Where to next

- **[Architecture](architecture.md)** — the packages, the reuse-vs-build decision,
  and why prior pure-Go browsers were studied but not built on.
- **[Roadmap](roadmap.md)** — the Phase 0–3 plan from static render to
  browserproxy.
- **[Fidelity](fidelity.md)** — an honest account of what renders correctly today
  and what does not.

## Links

- Source: <https://github.com/go-webengine/engine>
- Landing: <https://go-webengine.github.io/>
- License: BSD-3-Clause
