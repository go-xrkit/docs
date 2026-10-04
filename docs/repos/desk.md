# `desk` — screens on a band, inside AR glasses

[github.com/go-xrkit/desk](https://github.com/go-xrkit/desk)

Several computer screens on a band inside AR glasses, scrolled from the
keyboard. Pure Go, `CGO_ENABLED=0`, no vendor SDK.

XR glasses show one screen. This puts a row of them beside each other: real
virtual displays that macOS extends the desktop onto, so ordinary applications
run on them — the cursor goes there, you click in them — captured and drawn flat
on a band at eye level.

## The screens are flat

They were curved once, wrapped on a cylinder. Worn, that buys nothing: the screen
you look at straight on is drawn **with a bow in it**, and the bow argues with the
depth the glasses are already presenting. It also cost a projection — an
equirectangular panorama and a per-pixel warp, **2.8 ms of a 16.6 ms frame** — to
produce a picture whose whole purpose is to look like a flat screen.

So the screens are laid side by side on a flat band and the band slides.

⚠ **The BAND is flat; the screens can turn.** A plan carries a splay and a bend,
and a positive splay builds a renderer of turned facets instead. What never came
back is the WARP: no equirectangular panorama, no per-pixel distortion table. And
the flat band won the sharpness argument again on 2026-10-04 — see the wide
screens below.

**One screen is one full view, in PIXELS.** Each virtual display is created at
exactly one eye's resolution — the most the glasses can show at once — and drawn
at **one source pixel per panel pixel**. That is what makes it fill the glasses,
and it needs the panel's resolution and nothing else.

Which has a consequence worth stating plainly: **the field of view is reported,
not required.** It used to decide the layout, so a headset nobody had measured
could not be planned for at all and this refused rather than guess. It decides
nothing now. `xrdesk` runs on any glasses, including ones the catalogue has never
heard of, and says so — the model where it knows one, the display's own name
where it does not, and *"field of view not known"* rather than `0.00°`, because a
number is a claim.

**How many screens is a choice, and it stops at nine.** A curved band had to fit
in 360°, which at one screen per view was seven of them and no more. Flat, the
circle is a fiction: the yaw says how far along the band you are, and the band is
however long it needs to be — the plan would spread forty over the turn without
complaint.

So the limit is *decided*, at `desk.MaxScreens` = **9**, where three things
agree: the gallery is three columns wide, so nine fills three rows of three
exactly and every screen keeps its column as the desk grows; each screen costs a
display to create and a stream to capture; and there are nine digits for the jump
keys and none spare. Three, six or nine are the usual ones. Six by default.

## One screen wider than the glasses

A spreadsheet does not fit a screen the size of one eye's view. `-wide 6400`
makes every screen on the band that wide — three and a third views — and you
reach the rest of it by **turning your head**.

```
xrdesk -wide 6400 -screens 3 -reach 15
```

**The band stays flat, and that is the point.** The screens are drawn two ways:
the *strip* slides a flat band in row copies, the *fan* projects turned facets
one destination column at a time, and a projection resamples. Measured, source
columns per destination column in the middle third of the view, three screens of
6400 at a reach of 15°:

| head turned | flat band | bent band |
| --- | --- | --- |
| 0° | 1.000 | 1.000 |
| 5° | 1.000 | 1.019 (0.000 … 2.000) |
| 10° | 1.000 | 1.073 |
| **15°** | **1.000** | **1.167** (0.000 … 2.000) |
| 20° | 1.000 | 1.073 |
| 30° | 1.000 | 1.000 |

A rate of 0 is a source column drawn twice; a rate of 2 is one never drawn at
all. **At rest the two are identical** — which is why it took wearing to find,
and why the test that keeps it honest turns the head rather than rendering one
frame. So a wide desk takes the flat band unless you ask for something else.

### The curve is virtual

A flat screen of 6400 was unusable at its edges before this: a flat plane keeps
its edges further from the eye than its middle, and a rectilinear projection
draws them smaller for it — **28%** of the middle's scale at the edge of a 6400
screen, against **81%** at the edge of one that is a single view wide. Bending
the screen was the first remedy, and it is the one that cost the sharpness above.

The second remedy is to make the HEAD go further. `-reach` is the radius of a
curve that is never drawn: turn your head twenty degrees and the band moves as
though you had turned across a screen on a cylinder of that radius, so the part
in front of you is always square on, always at one source pixel per output pixel,
and nothing is lost to a fold. `desk.MinReachDeg` = 5 is the floor — five degrees
on the widest screen the band allows is already a gain of twelve, and below that
a head that twitches throws the picture across the band.

It is amplified by what the screen demands: `desk.ComfortableYawDeg` = 20 is the
turn a person makes without thinking, and a screen needing more than that gets
the difference as gain rather than asking for a bigger turn.

### Pushing it back moves both axes

`⌃⌥⌘-` and `⌃⌥⌘=` move the band away and back, between `desk.MinDistance` = 1
and `desk.MaxDistance` = 4 screens across the view. A distance scales the band in
**both** axes — it used to scale only its length, which made a pushed-back screen
half as wide and full height, and that is a squash rather than a distance.

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

It is **three columns wide** whenever three columns hold every screen in three
rows or fewer — which is exactly the three, six and nine a desk is usually built
from. A fixed width is the point: a screen keeps its COLUMN as others are added,
so the map you build of where things are survives the desk growing.

| screens | gallery |
| --- | --- |
| 3 | 3x1 |
| 6 | 3x2 |
| 9 | 3x3 |

Past nine a fixed three columns stops paying — fourteen rows of them is not a
gallery — and the shape is chosen instead by what leaves the screens biggest: for
each column count the rows follow, the cell size follows from the view and the
gap, and each screen keeps its own aspect ratio in its cell. Asking for a width
that does not fit is a preference and not a refusal: a gallery in the wrong shape
beats no gallery at all.

A ragged last row is left-aligned. A screen that moves sideways when another one
is added is a screen you have to look for.

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

The gallery is laid out in the **view** — the picture the glasses are actually
given — because a screen laid out anywhere else is a screen the viewer cannot
see, and seeing them all at once is the whole point.

## System-wide shortcuts

The point of a desk in glasses is that you are *using* the screens on it. A
shortcut that only worked when `xrdesk` was frontmost would be a shortcut for
nothing: the viewer is reading inside one of the screens and wants the next one,
without first clicking on a window they cannot see. So **forty** of them are
claimed system-wide and work while another application has the keyboard — the
desk's own window is deliberately passive and never takes the keyboard from what
is running on the screens.

| | |
| --- | --- |
| `⌃⌥⌘1`…`⌃⌥⌘9` | straight to that screen |
| `⌃⌥⌘0` | fit: one screen, the largest these glasses can show it |
| `⌃⌥⌘←` `⌃⌥⌘→` | turn the band |
| `⌃⌥⌘-` `⌃⌥⌘=` | move the band away, and back |
| `⌃⌥⌘F3` | show the SCREENS |
| `⌃⌥⌘F4` | show what is RUNNING on them |
| `⌃⌥⌘↩` | choose |
| `⌃⌥⌘F` | follow your head |
| `⌃⌥⌘C` | what this screen shows |
| `⌃⌥⌘X` | one application per screen |
| `⌃⌥⌘[` `⌃⌥⌘]` | flatten the screens, turn them |
| `⌃⌥⌘M` `⌃⌥⌘H` | the pointer here, and back to this Mac |
| `⌃⌥⌘D` | 3D on or off |
| `⌃⌥⌘P` `⌃⌥⌘L` | a picture of the view, a photograph of the room |
| `⌃⌥⌘F1`…`⌃⌥⌘F12` | the glasses themselves: brightness, volume, the room, down |
| `⌃⌥⌘S` `⌃⌥⌘⎋` | the settings, and quit |

The desk's README carries the full table, and a test there reads it and refuses a
row the code does not grant — that table had four wrong rows and sixteen missing
before the test existed.

⚠ **One prefix for all of them.** The band was on `⌥⌘←`/`⌥⌘→` once; somebody who
had learnt the desk pressed `⌃⌥⌘←` and got nothing, and one prefix everywhere
beat two keys saved.

⚠ **And the galleries are on function keys**, which took three tries. `⌃⌥⌘A` was
granted without complaint and never delivered a single press — an application's
own menu key is invisible to every check there is. The arrows were next, and they
are the pair a settings file most often wants for the distance. A function key
collides with neither. `Space` is bound to nothing at all.

This goes through [`go-macos/hotkey`](https://github.com/go-macos/hotkey), which
uses Carbon's `RegisterEventHotKey`. That matters for one reason: **it asks for
no permission at all** — no Accessibility prompt, no input monitoring, no trip to
System Settings. The two obvious alternative routes both demand the Accessibility
(TCC) grant.

`-no-global` declines the whole thing.

### They are not always the keys you get

Three things move a shortcut, and a session **prints what it landed on**.

**You moved it.** Every one of them can be re-bound from `desk.hcl`, except quit
— the way out of a desk that covers a display, which somebody wearing glasses
cannot reach through a menu bar.

**Something else holds it.** The shortcut then climbs a ladder of MODIFIERS —
`⇧⌥⌘`, then `⌃⇧⌥⌘` — keeping the key and changing what is held with it. A whole
GESTURE lands on one rung or none: pushing the band away and bringing it back are
one idea, and granting them on different modifiers reads as a broken application.
Measured — `⌃⌥⌘↑` pushed the band away sixteen times in one session while
`closer` sat alone on `⌃⌥⇧⌘↓` and never fired once.

**The key is elsewhere on your keyboard.** A claim is made for the key whose
LEGEND matches, not for the ANSI position, so `⌃⌥⌘=` is the key printing `=` on
an AZERTY keyboard. The start-up line names both when they differ:
`fit: ⌃⌥⌘= (asked for ⌃⌥⌘-, it was taken)`.

Whatever it lands on is **printed at start-up**, and that is not politeness — it
is required, because of the three ways a shortcut can already be taken, only two
are detectable:

| How it is taken | Detected? |
| --- | --- |
| Another Carbon hot-key holder | **yes** — registration returns `eventHotKeyExistsErr` (-9878) |
| A macOS system shortcut | **yes** — the candidate is checked against the system's symbolic hot keys *before* registering |
| An application's own menu key | **no** — invisible to everything |

That third row is why the claimed combination is shown rather than merely logged.
It is also the row that moved the galleries twice: `⌃⌥⌘A` was granted and never
delivered. And the band keys are *also* Safari's tab navigation — while `xrdesk`
runs it wins them and Safari quietly stops seeing them. That is
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
| [`go-xrkit/xrkit`](xrkit.md) | `glasses` names the headset; `ribbon` places the screens and its `Nav` drives the band, the gallery and the promoted screen |
| [`go-macos/accessibility`](https://github.com/go-macos/accessibility) | moves an application's windows onto a chosen screen |
| [`go-macos/hotkey`](https://github.com/go-macos/hotkey) | the system-wide shortcuts, through Carbon, with no permission at all |
| [`go-macos/virtualdisplay`](https://github.com/go-macos/virtualdisplay) | creates the displays macOS extends onto |
| [`go-macos/screencapture`](https://github.com/go-macos/screencapture) | streams their pixels |
| [`go-widgets`](https://github.com/go-widgets) | the window, and every pixel of interface |

There is no distortion table, no panorama and no projection. The screens are
flat, so the buffer they are composited into IS what the glasses are given: a
frame is a run-length copy of the band's window, and turning the band moves that
window along it. The 2.8 ms the warp used to cost is not spent.

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

This section says what was CONNECTED, as against what was read off a
specification sheet. A field of view taken from a data sheet renders everything in
the wrong place if the data sheet is wrong, and nothing about the picture says so.

| | |
| --- | --- |
| **VITURE Beast** | connected over DisplayPort and **rendered to**, six flat screens, with an application moved onto one of them and its position read back |
| **VITURE Luma Ultra** | rendered to, and its display name is the bare word `VITURE` — a BRAND, which the catalogue holds with no optics at all. The bus supplies the model |
| **XREAL One S** | **enumerated over USB only** — `3318:043e "XREAL 1S"`, identified by product id, on three ports across two buses. DisplayPort alternate mode never engaged: USB 2.0 alone, no second display at any layer. Its modes and its rendering are unconfirmed |

Capture of a whole display on macOS **is** now proven; it was blocked for a while
on a Screen Recording permission that could not be granted on the machine it was
built on.

Not proven: the permission dialogue for the Accessibility grant, and the
behaviour when that grant is absent — the machine this was built on holds it, so
the refusal path is written and reviewed but never exercised end to end.

## Putting applications on the screens

A ribbon screen is a **real desktop**. The cursor goes there, you click in it, a
window moved there stays there. So an application is put on one by moving its
windows, which on macOS is the Accessibility API's job and needs its grant —
said plainly rather than silently placing nothing.

It is written in the settings, because while you are wearing the glasses you
cannot see the monitor you would have to drag a window off:

```hcl
place "Safari"   { screen = 1 }
place "Terminal" { screen = 2 }
```

The name is a fragment, case-insensitively: you write `code`, not `Visual Studio
Code`. Screens are counted from 1 in the plan's own order, so nobody has to know
what a `CGDirectDisplayID` is. One application missing does not cost the others —
a desk of six where one is not running places the other five and says which one it
did not.

**A window told to fill a panel does not land where it was told.** macOS reserves
the top of a display for the menu bar and clamps a window out of it, so a window
sent to a 1920x1200 screen arrives at `1920x1169`, thirty-one points down. The
allowance for that is 64 points — a menu bar at any scale, and *smaller than half
a screen*, which is what makes it safe rather than lax: a window within it is
centred on the screen it was sent to and cannot be centred on a neighbour.

## Settings

HCL, not JSON, for the two reasons anyone gives when asked: it takes **comments**,
so a file can say *why* a shortcut was moved — the only part worth reading six
months later — and it has a **schema**, so a typo is a diagnostic naming the line
rather than a field silently left at its zero value.

It lives beside the glasses catalogue — `~/Library/Application
Support/go-xrkit/desk.hcl` on macOS, `~/.config/go-xrkit/desk.hcl` on Linux, or
wherever `$XRDESK_CONFIG` points — and **does not have to exist**.

```hcl
shortcut "gallery" { keys = "option+command+space" }
fallback = ["control", "shift", "control+shift"]

ribbon  { screens = 6, immersive = true }
glasses { model = "VITURE Luma Ultra" }
```

`option+command+left`, `Option-Command-Left` and `⌥⌘←` are the same combination,
in any order and any case. A combination with **no modifier** is refused: claimed
system-wide, a bare key is taken from every application on the machine, including
whatever you are typing into.

`xrdesk -settings` opens a window for all of it, and it opens **by itself** when
several headsets are attached and nobody has said which — once. Naming a display
with `-screen` is the answer for a script; writing a model in the settings is the
answer for somebody who has already chosen.

### `immersive`

macOS draws a menu bar on **every** display when Spaces are separate, so the
glasses carry one of their own — thirty points tall on a Beast — at a window level
above an ordinary window. It therefore lands *on top of* the picture, and the
desktop being shown in that picture has a menu bar already: two of them was the
first thing anyone noticed wearing this.

So the picture is put above that bar and the Dock. Measured on this machine:

| | layer |
| --- | --- |
| an ordinary window | 0 |
| the Dock | 20 |
| the menu bar | 24 |
| the ribbon | **25** |

The bar on your other screen is untouched — it belongs to a different display.
Turn `immersive` off if the glasses are your **main** display, where that bar and
the Dock are the real ones rather than a copy on a screen nobody is looking at.
