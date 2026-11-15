---
title: "depth3d — one picture, two eyes"
weight: 55
---

# `depth3d` — one flat picture in, two eyes out {#depth3d-one-flat-picture-in-two-eyes-out}

[github.com/go-xrkit/depth3d](https://github.com/go-xrkit/depth3d)

```go
c, err := depth3d.New(depth3d.Options{Model: "Depth.mlpackage", MaxShift: 24})
defer c.Close()

err = c.Convert(left, right, stride, src, srcStride, w, h)
```

It is the same job whether the picture is a film frame or a captured desktop,
and both applications here want it:
[`player`]({{< relref "player.md" >}}) converts a flat film as it plays, and
[`desk`]({{< relref "desk.md" >}}) converts whatever the glasses are showing.
**The second of those is why this is a package rather than a file inside one of
them** — the alternative was the same synthesis written twice, diverging.

## Two paths, and it always says which one ran {#two-paths-and-it-always-says-which-one-ran}

| | depth from | both views from | needs |
| --- | --- | --- | --- |
| accelerated | a real network on the **Neural Engine** ([`go-macos/coreml`](https://github.com/go-macos/coreml)) | compute kernels on the **GPU** ([`go-macos/metal`](https://github.com/go-macos/metal)) | macOS, a GPU and a Core ML depth model |
| portable | cues in the picture itself ([`go-images/depth`](https://github.com/go-images/depth)) | the processor | nothing at all |

The repository reports **about 0.4 ms of processor time** per frame on an
M4 Max for the accelerated path — which is what lets a compositor keep the rest
of the machine. The portable path needs no model, runs everywhere including a
browser, and is **visibly not as good**.

`Describe` reports which one is running, and a fall back is **always** logged. A
converter that quietly took the cheap path would look identical from the outside
except for being worse — the same reasoning the rest of this organisation
applies to a ribbon that silently carries the screens a machine already has.

## The two eyes may be one frame {#the-two-eyes-may-be-one-frame}

`left` and `right` may point into the **same** buffer — the two halves of a
side-by-side frame, addressed by that frame's stride — or into two separate
pictures. `player` wants the first, `desk` the second, and neither pays a copy
for the other's preference.

## What it cannot do {#what-it-cannot-do}

**It cannot invent what the camera never saw.** Where a near object moves aside,
what is behind it is guessed from the same row, so an edge is a little smeared.
That is the honest cost of the effect, and it is in the API's documentation
rather than in a footnote.

`Curve` is worth reading before turning on. An S-curve flattens both ends of the
depth range and gives the middle their relief, so the subject stands out — it is
**not** a comfort control, and a near object gets slightly *more* disparity. At
a comfortable `MaxShift` it is swamped anyway: twelve pixels each way is
thirteen distinct shifts for the whole range.

## How it is tested, including the part a build machine cannot run {#how-it-is-tested}

The portable layer — sizing, defaults, the fall back and the synthesis on the
processor — is held at **100%**, gated in CI. The accelerated path needs a GPU
and a depth model, which a build machine has neither of, so those tests skip and
say so:

```
XRKIT_TEST_MODEL=/path/to/Depth.mlpackage go test ./...
```

Two details are worth repeating because they are the kind of thing a green suite
usually hides:

- The GPU kernels are **a string compiled at run time**, so nothing in an
  ordinary build ever looks at them. A test compiles every one, with a negative
  control. The curve test binds a table and checks it reaches the kernel, again
  with a control — without it, "the curve changed the picture" could be a
  rebound allocation.
- The two implementations are checked **against each other**, not each against a
  tolerance: on a real photograph with a real network's depth map, the GPU
  synthesis and the portable one differ on **0 bytes out of 86 999 040**.
  Floating point would agree almost always, which is the worst kind of
  agreement, so the arithmetic is integer throughout and the curve travels as a
  table both sides index.

## Install {#install}

```
go get github.com/go-xrkit/depth3d
```

`CGO_ENABLED=0`.
