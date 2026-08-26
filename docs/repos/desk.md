# `desk` — screens on a 360° ribbon

[github.com/go-xrkit/desk](https://github.com/go-xrkit/desk)

Several computer screens, floating on a 360° ribbon inside AR glasses, scrolled
from the keyboard. Pure Go, `CGO_ENABLED=0`, no vendor SDK.

XR glasses show one screen. This puts a ring of them around you: real virtual
displays that macOS extends the desktop onto, so ordinary applications run on
them, captured and drawn as curved screens on a band at eye level.

**One screen is one full view.** Each virtual display is created at exactly one
eye's resolution — the most the glasses can show at once — and is given exactly
the arc that eye can see. Looking straight at a screen shows it edge to edge, at
one source pixel per output pixel. For a VITURE Beast in its side-by-side 3D mode
that is six screens of 1920x1080 at 51.57° each; for a Luma Ultra or an XREAL
One S, seven of 46.06°. The numbers are not configured — they follow from the
headset's optics.

## Why the keyboard

The Beast, the Luma Ultra and the XREAL One series all do their 3DoF anchoring
**inside the glasses**, which is also why their motion sensors are not offered to
the host. So head movement is theirs and the ribbon is yours: the two compose
instead of fighting. Turning the band is a key press, and it is deliberate.

## The keys

In the window, and in the glasses:

| Key | On the ribbon | In the gallery |
| --- | --- | --- |
| `←` `→` | turn the band | move the selection — it **wraps**, because the ribbon is a circle |
| `↑` `↓` | nothing: a band has no rows | move a row — it **clamps**, because the fold has no seam |
| `Enter` | — | go to the highlighted screen, the short way round |
| `Space` / `f` | fill the view with the focused screen | — |
| `g` | show every screen at once | put the ribbon back exactly as it was |
| `Tab` / `c` | show the next source here | — |
| `Escape` / `q` | quit | quit — a mode you cannot leave is a trap |

That `↑` `↓` do nothing on the ribbon is a decision, not an omission: a band has
no rows, and a key that did *something* there would have to invent a meaning the
viewer could not predict.

## The gallery

`g` opens a head-locked grid of every screen at once. It answers the question the
ribbon cannot: *where is the one three round the back?* — a screen behind the
viewer's head, which the band can only reach by turning through all the ones in
between.

It is worth its own mode only if every screen in it is big enough to
**recognise**, so the grid shape is derived rather than tabulated. For each
column count the rows follow, the cell size follows from the view and the gap,
and each screen is fitted into its cell keeping its aspect ratio; the shape
chosen is the one covering the most angular area. That is why six 16:9 screens in
a 16:9 view come out **3x2 rather than 6x1** — a row of six is limited by its
width to a sixth of the view, while 3x2 is limited by its height to a half. Ties
go to the shape wasting the fewest cells, then to the one with the fewest rows.

Three properties are deliberate and worth knowing:

- **The grid is rigid.** A ragged last row is left-aligned, not centred.
  Centring is prettier and it breaks the columns: `↓` from the top right would
  land between two cells, with no answer to "which one" a viewer could predict.
- **It is head-locked and stable.** Cells sit relative to straight ahead, so the
  gallery does not turn with the ribbon, and the order is always the ribbon's own
  order. A layout that moved every time it opened would cost the viewer the one
  thing a grid is for — knowing where a screen *was*.
- **Opening it costs nothing.** The geometry depends on neither the yaw nor the
  selection, so it is built once at start-up, like the compositor.

Opening the gallery selects the screen the viewer was already facing, not screen
zero. `Enter` restores the ribbon **first** and then aims, which is what makes
choosing the screen you were already looking at cost no motion at all, and any
other one a turn measured from where the viewer actually is. Pressing `g` again
puts the yaw, the target, the focus and the mode back exactly as the gallery
found them.

The gallery must fit inside the **view** — the arc the glasses really show — not
inside the panorama window, which is wider by a margin. A screen laid out in that
margin is a screen the viewer cannot see, and seeing them all at once is the
whole point.

## System-wide shortcuts

The point of a desk in glasses is that you are *using* the screens on it. A
shortcut that only worked when `xrdesk` was frontmost would be a shortcut for
nothing: the viewer is reading inside one of the screens and wants the next one,
without first clicking on a window they cannot see. So three of the keys are
claimed **system-wide** and work while another application has the keyboard:

| | |
| --- | --- |
| `⌥⌘←` `⌥⌘→` | turn the band |
| `⌥⌘Space` | show every screen at once |

This goes through [`go-macos/hotkey`](https://github.com/go-macos/hotkey) v0.1.0,
which uses Carbon's `RegisterEventHotKey`. That matters for one reason:
**it asks for no permission at all** — no Accessibility prompt, no input
monitoring, no trip to System Settings. The two obvious alternative routes both
demand the Accessibility (TCC) grant. Registering `⌥⌘←` on macOS 26.6.2 produced
no dialog.

`-no-global` declines the whole thing.

### They are not always the keys you get

`⌥⌘Space` is the Finder's search window on a stock macOS. So each shortcut goes
through a **fallback ladder** — the wanted combination first, then the same
combination with Shift added, then with Control, then with both. On the machine
this was built on the gallery ends up on `⌥⇧⌘Space`.

Whatever it lands on is **printed at start-up**, and that is not politeness — it
is required, because of the three ways a shortcut can already be taken, only two
are detectable:

| How it is taken | Detected? |
| --- | --- |
| Another Carbon hot-key holder | **yes** — registration returns `eventHotKeyExistsErr` (-9878) |
| A macOS system shortcut | **yes** — the candidate is checked against the system's symbolic hot keys *before* registering |
| An application's own menu key | **no** — invisible to everything |

That third row is why the claimed combination is shown rather than merely logged.
`⌥⌘←`/`⌥⌘→` register without complaint and are *also* Safari's tab navigation:
while `xrdesk` runs it wins them and Safari quietly stops seeing them. That is
the trade a global shortcut is, and it is printed rather than hidden.

A held key is **dropped rather than queued**. Presses arrive faster than a ribbon
can turn, and a queue of them would keep turning it long after the finger came
off.

### Not having them is not a reason not to run

macOS only, for now: Linux and Windows have no Carbon. There the keys work in the
window exactly as the table above says, and the run reports that the global ones
were not claimed. The same is true on a macOS desktop where every candidate on
the ladder is spoken for. A platform with no global shortcuts is a reason to run
without them, not a reason not to run.

## How it is put together

| | |
| --- | --- |
| [`go-xrkit/xrkit`](xrkit.md) | `glasses` names the headset and derives its field of view; `ribbon` places screens and composites them by yaw; `warp` turns the panorama into two eyes |
| [`go-macos/virtualdisplay`](https://github.com/go-macos/virtualdisplay) | creates the displays macOS extends onto |
| [`go-macos/screencapture`](https://github.com/go-macos/screencapture) | streams their pixels |
| [`go-widgets`](https://github.com/go-widgets) | the window, and every pixel of interface |

The renderer never rebuilds its distortion table. Building one costs 56.5 ms and
there are 16.6 ms in a frame — but on an equirectangular panorama a yaw is
exactly a horizontal shift, so the yaw is applied where the screens are
composited into the panorama, where it costs nothing at all.

## Platforms

| | Capture | Virtual displays |
| --- | --- | --- |
| macOS | ScreenCaptureKit | yes, via private CoreGraphics |
| Linux | X11, and Wayland where the compositor offers `wlr-screencopy` | no — capture the displays you have |
| Windows | in progress — enumeration must be added to `go-mswin/win32` first | no — an indirect display driver needs signing |
| Android | `MediaProjection` | **no — settled**; see [`android`](android.md) |

Where virtual displays are not available the ribbon carries the real displays and
windows instead. That is fewer screens, and everything else is identical — and it
says so, because a fallback that looks like the feature is worse than one that
announces itself.

## Coverage

The geometry, the plan and the compositor have no operating system in them and
are held at **100% statement coverage, gated in CI**. The gate selects files by
*shape* rather than by a list of names — everything that is not a platform file,
not a command and not a `_display.go` — so a new portable file is gated the day it
is written instead of the day someone remembers to add it.

Playback needs a display, a video file and a pair of glasses, none of which a
runner has, so a total-coverage figure would be a number chosen to pass rather
than a standard.

## What it does not claim

A VITURE Beast was connected over DisplayPort and rendered to. A Luma Ultra was
**enumerated over USB only**, so its display name and modes are unconfirmed.
Capture of a whole display on macOS is **not yet proven** — it is blocked on a
Screen Recording permission that could not be granted on the machine it was built
on; capture of an application window *is* proven.
