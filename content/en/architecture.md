---
title: "Architecture"
weight: 30
---

# Architecture {#architecture}

## Why this is its own module {#why-this-is-its-own-module}

An immersive video player needs orientation math, eye-packing rules and
projection geometry regardless of what decodes the frame or what draws the
window. None of it touches a GPU, a codec, or a display API, so it is
factored out into `go-xrkit/xrkit`: a pure-Go, zero-dependency module that
every package in it can be tested by pinning known directions — no headset,
no glasses, no display required.

```
                 ┌───────────────────────────────────────────────┐
   consumer  →   │              go-xrkit/player (xrplay)          │
                 │  hardware decode · window · orchestration      │
                 └───────────────────────┬────────────────────────┘
                                          │
                 ┌───────────────────────┴────────────────────────┐
   geometry  →   │                  go-xrkit/xrkit                 │
                 │      pose · stereo · projection · warp          │
                 └───────────────────────────────────────────────┘
```

## Boundaries {#boundaries}

- **Not a decoder.** Hardware video decode is
  [go-macos/avfoundation](https://github.com/go-macos/avfoundation); `xrkit`
  never touches a codec.
- **Not a windowing toolkit.** Putting pixels on the glasses' own display is
  [go-widgets/window](https://github.com/go-widgets/window).
- **Not an XR SDK, because there isn't one in this path.** XR glasses expose
  their 3D mode as an ordinary **display mode** — a VITURE Beast reports
  3840x1080 for side-by-side 3D and 1920x1200 for plain 2D. Stereo output is
  therefore a borderless window covering that display, with one eye's image in
  each half; `xrkit/stereo` is the arithmetic on the panel size that decides
  where each half is, not a binding to a headset vendor's runtime.

## How `go-xrkit/player` composes it {#how-go-xrkitplayer-composes-it}

`xrplay` joins three things that know nothing about each other:
`go-macos/avfoundation` for hardware decoding, `go-xrkit/xrkit` for the
geometry, and `go-widgets/window` for a full-screen window on the right
physical display. None of `xrkit` depends on the other two; `player` is where
they meet.

`Detect` reasons about a file's projection and stereo layout from its name
first and its frame shape second, and records its reasoning rather than
silently guessing — because a viewer that mis-detects 180° content as 360°
shows the world squeezed into half the view and gives the user nothing to
act on.

## Why head tracking comes from the CAMERA and not the IMU {#why-head-tracking-comes-from-the-camera-and-not-the-imu}

The VITURE Beast's IMU is not reachable over HID: its interfaces open without
complaint and `SetReport` reports success, and they emit nothing at all —
proven against a control run that captured 481 reports from the same reader
on other devices. See
[go-macos/iokit](https://github.com/go-macos/iokit).

So the orientation comes from the **camera** instead. `xrkit`'s `headflow`
matches column profiles between consecutive frames and recovers **yaw**;
`go-xrkit/android` feeds it from the glasses' UVC camera over `usbfs`. Measured
live on a Pixel 11 Pro Fold, the tracker costs **790 µs of an 88 ms interval —
0.9 %** and a worn left-right sweep comes back as **197.9° of travel**. The
calibration is a **field of view** rather than a column count
(`HorizontalFOV ≈ 72.45°`, `ColumnsPerDegreeAt(width)`), so the same constant
holds at any capture width.

⚠ That is **yaw only** — no pitch, no roll — and at ~42 ms a frame it is
comfortably inside the 107 ms that `headflow` puts on *choosing which screen
somebody faces*, and still outside the under-20 ms wanted for locking an image
to the head at 1:1.

That is also why `warp` is still a precomputed table rather than a per-frame
computation: with a fixed orientation the sampling is identical on every frame,
and the measured cost of the two-eye pass (~3 ms at 4K, ~8 ms at 8K, against a
16.6 ms budget at 60 Hz) means a GPU warp is not needed until the orientation
updates per frame — which it does not yet do in this path.

## Quality bar {#quality-bar}

- **`CGO_ENABLED=0`** across the org; no XR SDK, no native shims.
- **Zero third-party dependencies** — `xrkit`'s `go.mod` names nothing else.
- **100% statement coverage**, including error branches, gated in CI.
- **Six architectures** — build + test on amd64 / arm64 natively and
  loong64 / riscv64 / ppc64le / s390x under qemu. This is float64 geometry
  leaning on `Atan2`/`Asin`/`Acos` at their domain edges, and s390x is
  big-endian with a different FPU implementation, so these lanes gate
  platform-dependent floating-point behaviour rather than just compiling.
