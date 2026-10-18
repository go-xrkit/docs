---
title: "Roadmap"
weight: 90
---

# Roadmap {#roadmap}

## Shipped {#shipped}

**`go-xrkit/xrkit`** — `pose`, `stereo`, `projection`, `warp`, `ribbon` and
`glasses` are complete and gated at 100% statement coverage, on six
architectures. One dependency: HCL, and only for the user glasses catalogue.

**`go-xrkit/player`** (`xrplay`) — plays 360°, VR180 and 3D film files on XR
glasses: detects projection/layout from the file, decodes on the GPU via
[go-macos/avfoundation](https://github.com/go-macos/avfoundation), warps each
eye through `xrkit`, and shows the result full screen on the glasses' own
display via [go-widgets/window](https://github.com/go-widgets/window).

**`go-xrkit/desk`** — several computer screens on a flat band inside AR glasses,
scrolled from the keyboard. Real virtual displays that macOS extends the desktop
onto — the cursor goes there, you click in them — captured and drawn flat, with
applications moved onto them by name.

**`go-xrkit/android`** — screen capture on Android as an ordinary
`CGO_ENABLED=0` Go binary talking to a small Java host over a socket, plus a
documented account, with transcripts, of the four different ways Android 15
refuses to let an app create a display it can launch anything onto — **and the
glasses' UVC camera over `usbfs` on an unrooted phone**, at 23.8 fps with
nothing dropped.

## Known limits, today {#known-limits-today}

- **Head tracking is yaw, from the CAMERA, and not from an IMU.** The VITURE
  Beast's IMU is still not reachable over HID (see
  [Architecture]({{< relref "architecture.md" >}})), and newer generations use USB control
  transfers instead, which remains unstarted. What exists instead is
  `xrkit`'s `headflow`: it recovers **yaw** by matching column profiles between
  camera frames, measured live on a Pixel 11 Pro Fold at **790 µs of an 88 ms
  interval — 0.9 %**, and a worn left-right sweep comes back as **197.9° of
  travel**. That is enough to choose which screen somebody faces, which
  `headflow` itself puts at 107 ms; it is **not** pitch, not roll, and not the
  under-20 ms budget for locking an image to the head at 1:1.
- **No Matroska.** AVFoundation does not demux MKV or WebM; MP4/MOV/M4V play.
- **No seeking, no audio, no loop.** A file plays through once, silently.
- **macOS only.** The geometry and display logic in `xrkit` are portable and
  tested everywhere it builds; the decoder and window back-ends in `player`
  are not written for Linux or Windows yet.

## Next {#next}

- USB-control-transfer head tracking for newer XR glasses generations, which
  would put a GPU warp back on the table for per-frame orientation updates —
  the current CPU table only holds because orientation is fixed.
- Linux/Windows decode + window back-ends for `player`, reusing the same
  `xrkit` geometry unchanged.
