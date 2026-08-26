# `android` — capture, and displays you own

[github.com/go-xrkit/android](https://github.com/go-xrkit/android)

Screen capture on Android from **pure Go**, `CGO_ENABLED=0`, for an XR compositor
that redraws every frame — plus a Go API for the displays an ordinary app is
allowed to make for itself.

It is the Android member of a family —
[`go-macos/screencapture`](https://github.com/go-macos/screencapture),
[`go-freedesktop/screencast`](https://github.com/go-freedesktop/screencast),
[`go-mswin/screencapture`](https://github.com/go-mswin/screencapture) —
deliberately shaped so one consumer drives all of them through near-identical
adapters: `Displays`, `CaptureDisplay`, `Stream.Frame() (Frame, bool)` lending
borrowed pixels with a stride carried and never assumed, and an idempotent
`Close`.

## Two processes, because JNI needs cgo

Android hands no drawable surface to a process that is not the app, and every
path to one is behind JNI, which needs cgo. So an Android build is **two
processes**: a small Java host owning the Activity and the Surface, and an
ordinary `CGO_ENABLED=0 GOOS=android` Go binary speaking to it over a socket,
with pixels through a shared buffer. That is the pattern
[`go-widgets/android`](https://github.com/go-widgets/android) already proved with
a real installable APK.

`arm64` is the only CGO-free Android architecture, and CI asserts it: it checks
that `arm64` links CGO-free **and** that `arm`, `amd64` and `386` still require
cgo, so the day that changes it is noticed rather than assumed.

## What Android will and will not allow

The interesting question for an XR virtual desktop is not "can I capture the
screen" — it is "can I create extra desktops and put applications on them", the
way [`desk`](desk.md) does on macOS. The boundary is **not** *your own app*
versus *other people's*, and **not** *virtual display* versus *real one*. It is
an activity **launch** versus a **window**, on a display you *made* versus a
display you were *given*:

| | a display you were **given** (external, overlay) | a display you **made** (`createVirtualDisplay`) |
| --- | --- | --- |
| **launch an activity** on it (`setLaunchDisplayId`) | yes — ours *and* Settings | **no** — `SecurityException`, even for our own activity |
| **show a `Presentation`** on it | yes | **yes**, and the pixels arrive |

So an unprivileged APK **cannot manufacture desktops for other applications**.
Asked four ways, refused four ways: a virtual display comes back without
`FLAG_TRUSTED`; `VIRTUAL_DISPLAY_FLAG_TRUSTED` wants `ADD_TRUSTED_DISPLAY`, whose
protection level is `signature` and which is not in the public SDK at all; and
`VirtualDeviceManager` needs `CREATE_VIRTUAL_DEVICE`, which is `internal|role`
and not grantable even by signature.

But it **can** manufacture several hundred independent displays of its own
content, each rendered by the real Android view system and each read back as
pixels — with no permission, no consent dialog and no screen-recording chip in
the status bar. That is a ring of panels carrying what Android can draw and a Go
process cannot: a WebView, a decoder, a map. It is a great deal more than one
mirrored phone, and it is what the Android ribbon is built on.

On the display the glasses themselves provide — public and trusted, as any real
external display is — an ordinary app **may** place other applications with
`ActivityOptions.setLaunchDisplayId`. The refusal is about *making* a display,
not about using the one you were given.

## The limit is the safety

There is a ceiling on how many displays may be made at once, and past it the
system dies. The ceiling observed on one emulator was **304**, and that number is
deliberately **not hard-coded anywhere** — it is one emulator's figure. The
*shape* of the failure is the finding, and the limit refuses in both directions
rather than trusting a constant.

## Coverage

100% statement coverage, gated in CI, error branches included. The whole test
suite runs on all six of Go's 64-bit targets, the four non-native ones under
qemu.

## What it does not claim

**No XR glasses were ever attached to an Android device for any of this.**
Everything was measured on an arm64 Android 15 (API 35) emulator, and the
transcript *is* the documentation: the probe that produced every number is in the
APK and can be re-run.

If you want a figure measured rather than quoted, **send us the hardware**.
