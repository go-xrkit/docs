# `player` — immersive video on the glasses

[github.com/go-xrkit/player](https://github.com/go-xrkit/player)

`xrplay` plays immersive video on XR glasses: 360°, VR180 and 3D films, per eye,
full screen on the glasses' own display. `CGO_ENABLED=0`.

```
xrplay film.mp4                       # detect everything, find the glasses
xrplay -proj 360 -layout sbs f.mp4    # override the detection
xrplay -screen "Built-in" -mono f.mp4 # one eye on the laptop, to look at it
xrplay -for 10s -snapshot out.png f   # stop after 10s, capture what was shown
```

It joins three things that know nothing about each other:
[`go-macos/avfoundation`](https://github.com/go-macos/avfoundation) for hardware
decoding, [`xrkit`](xrkit.md) for the geometry, and
[`go-widgets/window`](https://github.com/go-widgets/window) for a full-screen
window on the right physical display.

## There is no SDK in the stereo path

XR glasses expose their 3D mode **as a display mode**: a VITURE Beast reports
3840x1080 for side-by-side 3D and 1920x1200 for ordinary 2D. So stereo output is
a borderless window covering that display with one eye's view in each half —
which is all `StereoMode` decides, from arithmetic on the panel size.

The blind spot is documented rather than hidden: a genuine 32:9 monitor has the
same aspect as two 16:9 eyes, and no arithmetic can separate them. Hence
`-screen` and `-mono`.

For image quality, put the glasses in their **pixel-exact** mode. macOS also
offers scaled modes — a Beast reporting "5120x1600, looks like 2560x800" is being
rendered at 5120x1600 and downsampled onto a 3840x1080 panel, which costs
sharpness for nothing.

## What it guesses, and why it says so

None of a file's immersive geometry is reliably recorded in its container, and
the conventions that exist are conventions of *file naming*. `Detect` reasons
from the name first and the frame shape second, and **records its reasoning**:

```
content   equirectangular 360x180, mono eyes
  because: no stereo marker in the name and no stereo aspect; 2:1 eye, so equirectangular 360
```

A viewer that silently mis-detects 180° content as 360° shows the world squeezed
into half the view and gives the user nothing to act on. Every axis is
overridable with `-proj`, `-layout` and `-swap`.

One ambiguity is worth knowing: real VR180 stores 180°x180° per eye, which is
**square**, so a side-by-side VR180 frame is 2:1 overall — indistinguishable by
aspect from a monoscopic sphere. Only the name tells them apart.

## Opening is not working

A source is accepted only once it has actually **produced a picture**. Opening
succeeds on files that never decode, so opening is not the test.

## Controls

The transport bar appears when the pointer moves and hides after three idle
seconds. Every element is a go-widgets widget and every glyph an Iconoir drawing
through `Button`'s icon seam — nothing paints a pixel of its own, because a
hand-drawn bar is a private set of shapes that no theme, no HiDPI scale and no
accessibility walk knows about.

In a side-by-side 3D mode the bar stays in the **left eye's half**: drawn across
the panel it would appear twice, at different depths, and read as a double image.
The pointer has to be over the glasses' display for the bar to appear; the
keyboard works wherever the pointer is.

| Key | |
| --- | --- |
| `space` / `k` | pause, resume |
| `←` `→` / `j` `l` | seek 10 s |
| `↑` `↓` | volume |
| `Esc` / `q` | quit |

Both sets are bound: `j`/`k`/`l` is what every video player has taught people,
and the arrows are what everyone tries first. An unknown key does **nothing** — it
used to quit, which meant brushing the keyboard closed the film, and a viewer in
a headset cannot see what they pressed.

The sound is the clock.

## Limits

- **No loop.** A file plays through once. Seeking needs a clocked source, so it
  works on the AVPlayer path and not on the demuxed one.
- **AV1, VP9 and VP8 in Matroska are not decoded** — H.264 and HEVC only.
- **No head tracking**, because the glasses keep their IMU to themselves.
- **macOS only.** The geometry and display logic are portable and tested
  everywhere; the decoder and window back-ends are not written for Linux or
  Windows yet.
