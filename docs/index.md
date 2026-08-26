# go-xrkit documentation

**The geometry an immersive video player and an XR virtual desktop need, as
pure Go:** orientation, stereo frame packing, the projections that turn a flat
frame into a world you can look around in, the 360° band screens sit on, and
the catalogue that says how wide a given headset's view actually is.

`CGO_ENABLED=0`, one third-party module (HCL, and only for the glasses
catalogue — see [`glasses`](packages.md#glasses) for why), 100% statement
coverage, and every package testable without a headset attached — which is the point. Sign and
axis-order mistakes are invisible in a still frame and awful to wear, so they
are pinned by tests against known directions rather than discovered by putting
the glasses on.

```go
import (
    "github.com/go-xrkit/xrkit/pose"
    "github.com/go-xrkit/xrkit/stereo"
    "github.com/go-xrkit/xrkit/projection"
    "github.com/go-xrkit/xrkit/warp"
)

// Fixed orientation (no head tracking yet) → precompute the warp once.
q := pose.Identity()
vp := projection.Viewport{Width: 1920, Height: 1080, FOVyDeg: 90}
f := stereo.Format{Layout: stereo.SideBySide}

m := warp.Build(vp, projection.Sphere360, q, warp.Source{
    Width: srcW, Height: srcH, Stride: srcW,
    Eye: f.EyeRect(stereo.Left, srcW, srcH),
})

// Per frame: a gather, not a computation.
m.ApplySwapRB(src, dst, dstStride, 0, 0)
```

## The packages

| Package | Purpose |
| --- | --- |
| [`pose`](packages.md#pose) | quaternions, the Euler convention head trackers report, recentring, smoothing |
| [`stereo`](packages.md#stereo) | how a frame packs two eyes — mono, side-by-side, top-bottom |
| [`projection`](packages.md#projection) | flat / equirectangular (360×180, VR180 180×180) / equidistant fisheye |
| [`warp`](packages.md#warp) | the projection precomputed into a lookup table |
| [`ribbon`](packages.md#ribbon) | screens on a 360° band, composited by yaw |
| [`glasses`](packages.md#glasses) | which display is the headset, and how wide the view is |

## Where to go next

- [Packages](packages.md) — what each package provides, and the bugs their
  design choices exist to prevent.
- [Architecture](architecture.md) — why the geometry lives in its own module,
  and how `go-xrkit/player` puts it to use.
- [Repositories](repositories.md) — the four repositories in the org and what
  each one is for, each with its own page: [xrkit](repos/xrkit.md),
  [player](repos/player.md), [desk](repos/desk.md) and
  [android](repos/android.md).
- [Roadmap](roadmap.md) — what ships today and what is still missing.

Source: [github.com/go-xrkit/xrkit](https://github.com/go-xrkit/xrkit) · what
consumes it: [player](https://github.com/go-xrkit/player),
[desk](https://github.com/go-xrkit/desk) and
[android](https://github.com/go-xrkit/android).
