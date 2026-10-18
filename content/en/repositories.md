---
title: "Overview"
weight: 40
---

# Repositories {#repositories}

Four repositories with code: the geometry, and the three things built on it.

| Repository | What it is |
| --- | --- |
| [`xrkit`]({{< relref "repos/xrkit.md" >}}) | the geometry — `pose`, `stereo`, `projection`, `warp`, `ribbon`, `glasses`. Every package testable with no headset attached, gated at 100% coverage |
| [`player`]({{< relref "repos/player.md" >}}) | `xrplay film.mp4` — 360°, VR180 and 3D films, per eye, full screen on the glasses' own display |
| [`desk`]({{< relref "repos/desk.md" >}}) | several computer screens on a flat band, scrolled from the keyboard |
| [`android`]({{< relref "repos/android.md" >}}) | screen capture on Android and the glasses' UVC camera over `usbfs`, as a CGO-free Go binary talking to a small Java host |

## How they fit together {#how-they-fit-together}

`xrkit` knows no operating system at all. Everything platform-shaped is reached
through modules that belong to the platform's own organisation, and none of them
knows about any of the others:

| | |
| --- | --- |
| [`go-macos/virtualdisplay`](https://github.com/go-macos/virtualdisplay) | creates the displays macOS extends onto — on **private** CoreGraphics classes, which is stated plainly there |
| [`go-macos/screencapture`](https://github.com/go-macos/screencapture) | streams their pixels |
| [`go-macos/avfoundation`](https://github.com/go-macos/avfoundation) | hardware decode for `player` |
| [`go-freedesktop/screencast`](https://github.com/go-freedesktop/screencast) | the same capture shape on Linux, over X11 |
| [`go-mswin/screencapture`](https://github.com/go-mswin/screencapture) | the same shape again on Windows |
| [`go-widgets`](https://github.com/go-widgets) | the window, and every pixel of interface |

## What each one says it has NOT proven {#what-each-one-says-it-has-not-proven}

This is a deliberate convention across the org, not an accident of tone. Each
repository separates hardware that was connected and exercised from hardware
only partially observed, and both from what is known solely from a specification
sheet — and each invites you to send hardware so a quoted figure can become a
measured one.

- **`desk`** — a VITURE Beast was connected over DisplayPort and rendered to; a
  Luma Ultra was **enumerated over USB only**, so its display name and modes are
  unconfirmed. Capture of a whole display on macOS is **not yet proven**: it is
  blocked on a Screen Recording permission that could not be granted on the
  machine it was built on. Capture of an application window *is* proven.
- **`android`** — **glasses have been attached since 2026-10-18, for the camera
  and head-tracking rows only.** A VITURE Beast on a Pixel 11 Pro Fold: the
  camera streams through `usbfs` on an unrooted phone at **23.8 fps with nothing
  dropped**, and a worn head turn comes back as **197.9° of travel**. Everything
  else on that page — screen capture, virtual displays, the 304-display ceiling —
  is still an arm64 Android 15 **emulator's** figure. That ceiling is one
  emulator's number and must not be hard-coded anywhere; the *shape* of the
  failure is the finding. Why MJPEG delivers only 3.21 fps on that camera is
  **open**, and an earlier claim that the camera truncates its own frames is
  withdrawn — see [`android`]({{< relref "repos/android.md" >}}).
- **`xrkit`** — the `glasses` catalogue grades every entry `Observed`,
  `Enumerated` or `Published`, and no macOS display-name artifact has been read
  for any model yet.
