---
title: "android — capture and owned displays"
weight: 80
---

# `android` — capture, the glasses' camera, and displays you own {#android-capture-the-glasses-camera-and-displays-you-own}

[github.com/go-xrkit/android](https://github.com/go-xrkit/android)

Screen capture on Android from **pure Go**, `CGO_ENABLED=0`, for an XR compositor
that redraws every frame — plus a Go API for the displays an ordinary app is
allowed to make for itself, and a **UVC camera client** that reads the glasses'
camera through `usbfs` on an **unrooted** phone.

It is the Android member of a family —
[`go-macos/screencapture`](https://github.com/go-macos/screencapture),
[`go-freedesktop/screencast`](https://github.com/go-freedesktop/screencast),
[`go-mswin/screencapture`](https://github.com/go-mswin/screencapture) —
deliberately shaped so one consumer drives all of them through near-identical
adapters: `Displays`, `CaptureDisplay`, `Stream.Frame() (Frame, bool)` lending
borrowed pixels with a stride carried and never assumed, and an idempotent
`Close`.

## Two processes, because JNI needs cgo {#two-processes-because-jni-needs-cgo}

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

## What Android will and will not allow {#what-android-will-and-will-not-allow}

The interesting question for an XR virtual desktop is not "can I capture the
screen" — it is "can I create extra desktops and put applications on them", the
way [`desk`]({{< relref "desk.md" >}}) does on macOS. The boundary is **not** *your own app*
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

## The limit is the safety {#the-limit-is-the-safety}

There is a ceiling on how many displays may be made at once, and past it the
system dies. The ceiling observed on one emulator was **304**, and that number is
deliberately **not hard-coded anywhere** — it is one emulator's figure. The
*shape* of the failure is the finding, and the limit refuses in both directions
rather than trusting a constant.

## Coverage {#coverage}

100% statement coverage, gated in CI, error branches included — 87 functions,
100.0% under both `go1.26.4` and `go1.27.1`. The whole test suite runs on all six
of Go's 64-bit targets, the four non-native ones under qemu.

⚠ Both halves of that matter together. The gate is on the TOTAL with no margin,
and Go 1.27 counts more statements per block than 1.26 — 381 to 399 here on one
commit — so a toolchain change moves the number under it without a line of code
changing. It is harmless at exactly 100%: when every statement is covered the
weights cannot matter. And because the qemu lanes EXECUTE rather than
cross-compile, `arch loong64 (qemu)` is a real witness for golang/go#81000,
whose backport is still open — it passes under 1.27.1 here, where the same lane
fails in go-gfx/gfx. A code-generation defect is established by LANE, never by
repository.

## The camera: UVC over `usbfs`, on a phone nobody rooted {#the-camera-uvc-over-usbfs-on-a-phone-nobody-rooted}

Since v0.12.0 this package also reads the camera the glasses carry. There is no
`/dev/video*` for it — an ordinary Android app is refused both nodes — so the
stream goes through **`usbfs`**: the Java host calls `openDevice`, lends the file
descriptor over the socket, and the Go process drives UVC on it directly with
`USBDEVFS_*` ioctls whose numbers are **computed** from this package's own
mirrors of the kernel structs and asserted against `usbdevice_fs.h`.

Both formats are decoded, and they are not whole in the same way:

| | whole by |
| --- | --- |
| **YUY2** | its **length**, `w*h*2`. A short frame lost packets and is dropped, never padded |
| **MJPEG** | its **markers**. A compressor's output is whatever the scene made it, so "shorter than expected" describes a *wall*, not a loss |

`NewFrameAssemblerFor` takes the format for exactly that reason, and
`UVCNegotiation.Kind` is where it comes from — a negotiation result should say
what was *agreed* rather than leave a stream to guess.

### Measured on the device, with the conditions it was measured under {#measured-on-the-device-with-the-conditions-it-was-measured-under}

2026-10-18, a **VITURE Beast** on a **Pixel 11 Pro Fold**:

| | |
| --- | --- |
| **rate** | **23.8 fps with nothing dropped** at 640×480 YUY2, against the 25 the camera advertises; 14.8 and none at 320×240. ⛔ **This did not reproduce on 2026-10-31 — see below** |
| **a worn head turn, replayed offline** | **197.9° of travel** in 2.9 s — out to −52°, back **through zero**, out to +80° — 27 of 30 frames tracked by [`xrkit`]({{< relref "xrkit.md" >}})'s `headflow` |
| **live, in one process on the phone** | **748.5° of travel** over 86 matched frames in 8 s, the tracker itself costing **790 µs of an 88 ms interval — 0.9 %** |

⚠ **Those two rows were taken under different ring depths and must not be added
together.** The live run delivered 11.4 fps (an 88 ms interval) and dropped 101
frames for every 91 it kept; the 23.8 came afterwards, from the fix that run
exposed. A live tracking run at the new depth has not been taken.

⛔ **The frame rate was OURS, not the camera's, for five releases.**
`StreamURBs` — isochronous requests in flight — was **4**, covering 4 ms of bus
schedule, and any consumer costing more than that left nothing submitted while it
worked. Sixteen gives 10.5 → **23.8 fps** and 79 → **0** dropped on the same
phone. Thirty-two makes this camera stop sending altogether, which closed that
axis; so in **v0.17.0** the reaping became a goroutine that only reaps, assembles
and resubmits, and the decode moved to the caller's goroutine where its cost is
the caller's.

⚠ That paragraph is **half** of the story, and the half that reproduces is the
*shape* of the sweep, not its height — see
[23.8 fps did not reproduce](#the-rate-did-not-reproduce) below. The ring bound
the rate on the evening the camera had bytes to give; on an evening when it does
not, the ring is no longer what binds.

⚠ **MJPEG is slower here and WHY is open.** It advertises 30 fps at every size
and delivered 1.97 fps at a ring of 4, 3.21 at 16, with every loss showing a
start-of-image and no end-of-image. An earlier version of this page's upstream
README called that *"the camera truncates its own frames"*; that reading is
**withdrawn**. A missing tail says where the bytes stopped arriving, not who
stopped sending them.

## ⛔⛔⛔ 23.8 fps did not reproduce, thirteen days later {#the-rate-did-not-reproduce}

2026-10-31, same phone, same glasses, same commit, in the configuration that
produced it — default pick, `YUY2 640×480 @25fps`, payload 2400, alternate 5,
`StreamURBs = 16`. **Five runs: 9.97, 10.0, 9.98, 10.0, 9.97 fps**, every one
with **0 dropped and 0 late**.

The ring sweep, repeated the same evening, keeps its **shape** and loses half
its **height**: ring 4 → 8.91 fps and 2 dropped; ring 16 → 10.0 and none. So
`StreamURBs = 16` keeps its justification — it removes the dropped frames in
both regimes — while **"23.8 fps" turns out to be a figure of that evening's
camera state, not a property of this device.**

What differs is the camera's own byte rate for an identical negotiation: **781
bytes an interval** of the 2400 it reserved, and the total is *exactly* ten
614 400-byte frames plus 7 992 × 12 bytes of UVC header — so nothing is missing
and nothing is ours. The afternoon of 18 October it must have emitted ~1 830.

⚠ **It is not the light.** Two runs, luminance `4..86` and `37..255`, returned
**the same byte and packet totals** (6 239 904 / 7 992). YUY2 frames are a fixed
size, so its byte rate *is* its frame rate — and that did not move when the
scene brightened threefold. The obvious guess, refuted.

⭐⭐ **And MJPEG, twice the same evening, twenty minutes apart: 3.44 and
1.71 fps** — a factor of two on its own, which is why no rate comparison is
offered for it either. What *both* runs agree on needs no comparison: `0 late`
(the handover never shed a frame), ≈8 000 packets a second (**one per
microframe**, so the isochronous schedule was fully claimed) and 70 bytes an
interval of the 800 reserved. **Our own starvation is excluded by those
invariants**; why the camera truncates stays unexplained.

**Still open:** why the same camera gives 1 830 bytes an interval one day and
781 another. The glasses were **worn** for the 18 October runs and sat on a desk
for these — uncontrolled both times, so it is the next variable to try rather
than an explanation.

⛔ **The frames are pictures of a room and of whoever is wearing the headset.**
They go to the durable directory outside every work tree that
[`go-appdirs/outdir`](https://github.com/go-appdirs/outdir) chooses — never into
a repository, and a `.gitignore` entry would be the wrong fix — and the tool
prints the path so a person can look and delete.

## What it does not claim {#what-it-does-not-claim}

**The glasses are attached now, and what that changes is marked above.** Until
2026-10-18 nothing here had seen XR hardware; the capture and virtual-display
figures on this page are still an **arm64 Android 15 (API 35) emulator's**, and
only the camera and head-tracking rows above were measured on a real phone with
real glasses. The transcript *is* the documentation in both cases: the probe that
produced every number is in the APK and can be re-run.

If you want a figure measured rather than quoted, and it is not one of the rows
above, **send us the hardware**.
