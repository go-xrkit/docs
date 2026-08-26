# Roadmap

## Shipped

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
refuses to let an app create a display it can launch anything onto.

## Known limits, today

- **No head tracking.** The VITURE Beast's IMU is not reachable over HID (see
  [Architecture](architecture.md)); newer generations use USB control
  transfers instead, which is a separate, unstarted piece of work.
- **No Matroska.** AVFoundation does not demux MKV or WebM; MP4/MOV/M4V play.
- **No seeking, no audio, no loop.** A file plays through once, silently.
- **macOS only.** The geometry and display logic in `xrkit` are portable and
  tested everywhere it builds; the decoder and window back-ends in `player`
  are not written for Linux or Windows yet.

## Next

- USB-control-transfer head tracking for newer XR glasses generations, which
  would put a GPU warp back on the table for per-frame orientation updates —
  the current CPU table only holds because orientation is fixed.
- Linux/Windows decode + window back-ends for `player`, reusing the same
  `xrkit` geometry unchanged.
