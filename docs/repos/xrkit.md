# `xrkit` — the geometry

[github.com/go-xrkit/xrkit](https://github.com/go-xrkit/xrkit)

The geometry an immersive video player and an XR virtual desktop need, and
nothing else: orientation, how a frame packs two eyes, the projections that turn
a flat frame into a world you can look around in, the band screens sit on, and a
catalogue of what a given headset can actually show.

`CGO_ENABLED=0`. **100% statement coverage**, gated in CI. The geometry packages
have no dependencies at all; `glasses` uses HCL so a person can add their own
hardware without a rebuild.

Every package is testable with **no headset attached**, which is the point. Sign
and axis-order mistakes are invisible in a still frame and awful to wear, so they
are pinned by tests against known directions rather than discovered by putting
the glasses on.

## The six packages

Each is described in full on the [Packages](../packages.md) page.

| Package | Purpose |
| --- | --- |
| [`pose`](../packages.md#pose) | quaternions, the Euler convention head trackers report, recentring, smoothing |
| [`stereo`](../packages.md#stereo) | how a frame packs two eyes — mono, side-by-side, top-bottom |
| [`projection`](../packages.md#projection) | flat / equirectangular (360×180, VR180 180×180) / equidistant fisheye |
| [`warp`](../packages.md#warp) | the projection precomputed into a lookup table |
| [`ribbon`](../packages.md#ribbon) | screens on a 360° band, composited by yaw — and the gallery that shows them all at once |
| [`glasses`](../packages.md#glasses) | which display is the headset, and how wide the view is |

## Who consumes it

[`player`](player.md), [`desk`](desk.md) and [`android`](android.md). `xrkit`
knows about none of them, and knows no operating system at all.

## What it does not claim

The `glasses` catalogue grades every entry `Observed`, `Enumerated` or
`Published`, and an entry whose figure cannot be sourced carries none — because a
wrong field of view renders everything in the wrong place with no symptom. No
macOS display-name artifact has been read for any model yet.
