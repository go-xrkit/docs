# Repositories

Four repositories with code: the geometry, and the three things built on it.

| Repository | What it is |
| --- | --- |
| [`xrkit`](https://github.com/go-xrkit/xrkit) | the geometry — `pose`, `stereo`, `projection`, `warp`, `ribbon`, `glasses`. Every package testable with no headset attached, gated at 100% coverage |
| [`player`](https://github.com/go-xrkit/player) | `xrplay film.mp4` — 360°, VR180 and 3D films, per eye, full screen on the glasses' own display |
| [`desk`](https://github.com/go-xrkit/desk) | several computer screens on a 360° ribbon, scrolled from the keyboard |
| [`android`](https://github.com/go-xrkit/android) | screen capture on Android, as a CGO-free Go binary talking to a small Java host |

## How they fit together

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

## What each one says it has NOT proven

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
- **`android`** — **no XR glasses were attached to an Android device for any of
  it.** Everything was measured on an arm64 Android 15 emulator. The ceiling of
  304 displays is one emulator's number and must not be hard-coded anywhere; the
  *shape* of the failure is the finding.
- **`xrkit`** — the `glasses` catalogue grades every entry `Observed`,
  `Enumerated` or `Published`, and no macOS display-name artifact has been read
  for any model yet.
