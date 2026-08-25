# Packages

go-xrkit is a single Go module, `github.com/go-xrkit/xrkit`, split into
focused packages. Import only what you need; each is usable on its own and
depends on nothing third-party.

## pose

Quaternions, the Euler convention head trackers actually report, recentring
and smoothing.

```go
q := pose.FromEulerZXY(pose.Euler{Yaw: -30, Pitch: 10})
dir := q.Rotate(pose.Vec3{Z: -1})     // where the viewer is looking

r := pose.NewRecentre()
r.Set(q)                              // "this is straight ahead now"
rel := r.Apply(next)

s := pose.Smoother{Alpha: 0.35}       // a tracker is noisy at rest
smooth := s.Update(rel)
```

**Yaw is applied last.** `FromEulerZXY` composes roll, then pitch, then yaw, so
yaw stays a turn about the global up axis. Compose it first instead and
pitching to 90° no longer looks straight up: there is no gimbal lock, and the
horizon swings as the viewer raises their head. This package shipped that bug
once; every single-axis test passed while it did, because with one non-zero
angle the order cannot matter.

## stereo

How a frame packs two eyes.

```go
f := stereo.Format{Layout: stereo.SideBySide}
r := f.EyeRect(stereo.Left, 3840, 1080)   // {0, 0, 1920, 1080}
```

Layouts: mono, side-by-side, top-bottom. `Swapped` is an explicit flag, never a
guess: eye-reversed material is not detectable from the pixels, and getting it
wrong inverts the depth of the whole scene — which viewers report as eye
strain rather than as a wrong picture. An odd frame dimension floors the split
and leaves the middle line unread: losing one column is invisible, a column of
the wrong eye is not.

## projection

Direction ↔ picture.

```go
vp := projection.Viewport{Width: 1920, Height: 1080, FOVyDeg: 90}
dir := vp.LookRay(headOrientation, x, y)
u, v, ok := projection.Sphere360.Sample(dir)
if !ok {
    // outside the content — show background, do not clamp: clamping smears
    // the edge pixels across the whole of the missing region
}
```

Three projections: `Flat` (a virtual screen), `Equirect` (360×180, or VR180's
180×180) and `Fisheye` (equidistant, 180° to 200°). Fisheye is *equidistant*,
not tangent: radius is proportional to the angle from the axis. A tangent law
agrees at the centre and is wrong everywhere else — the kind of error that
looks plausible in a still.

Rays are taken through pixel **centres**. Sampling the corner instead biases
the whole image by half a pixel: invisible alone, a visible seam where two
views meet.

## warp

The projection precomputed into a lookup table, so reshaping a video frame for
one eye costs a copy rather than a computation.

```go
m := warp.Build(vp, projection.Sphere360, q, warp.Source{
    Width: srcW, Height: srcH, Stride: srcW,
    Eye: f.EyeRect(stereo.Left, srcW, srcH),
})
m.ApplySwapRB(src, dst, dstStride, 0, 0)
```

The per-pixel trigonometry in `projection` is fine to reason about and far too
slow for four million pixels sixty times a second. But it only depends on the
viewer's **orientation**, and when that is fixed — no head tracking, a screen
anchored in front of the viewer — the answer for every output pixel is the
same on every frame. So it is computed once into a table of source offsets,
and each frame becomes a gather. Nearest-neighbour sampling is deliberate: the
output of an immersive viewer is already a magnification of the source, so the
visible artefact is the magnification itself, not the interpolation.

A `Map` is tied to the geometry it was built for — orientation, output size,
projection, source dimensions — and must be rebuilt if any of those change.

**Measured**, reshaping both eyes at once: **~3 ms at 4K**, **~8 ms at 8K**,
zero allocations per frame — against a 16.6 ms budget at 60 Hz.

## ribbon

Screens on a 360° band: where each one sits, and how they composite into one
equirectangular panorama by yaw.

The reason this is a package and not a loop in an application is a performance
fact that decides the whole design. Building a `warp.Map` costs **56.5 ms**, and
there are **16.6 ms** in a frame at 60 Hz — so a ribbon whose yaw changed the
warp table could never be turned. But on an equirectangular panorama a yaw is
**exactly a horizontal shift**. So the yaw is applied when the screens are
composited into the panorama, where it costs nothing at all, and the distortion
table is never rebuilt.

`ribbon` also decides how many screens fit and how wide each one is, from the
headset's own optics rather than from configuration: each screen gets exactly
one eye's resolution and exactly the arc that eye can see, so looking straight
at one shows it edge to edge at one source pixel per output pixel.

## glasses

Which display is the headset, and how wide is the view.

Every entry in the catalogue records **how far it has been checked**, because
almost everything about a headset is read off a specification sheet, and a field
of view taken from a wrong data sheet renders everything in the wrong place
while nothing about the picture says so:

| Confidence | Meaning |
| --- | --- |
| `Observed` | connected, and its modes seen |
| `Enumerated` | seen on the bus, but its video was never connected |
| `Published` | sourced and cited, never plugged in here |

Identification is not one thing — a display name, a USB vendor/product pair and
a mode list each answer a different question, and the package keeps them
separate rather than pretending one implies another.

The catalogue will always be behind the hardware, so a model can be declared in
an HCL file under the platform's config directory, and a user entry wins ties
against the built-in one. HCL is the module's **one dependency**, and it is
there for a specific reason: a figure about a headset is only worth something
with its provenance attached, and HCL has *comments*, so what you measured sits
next to the number you measured it from. A file that exists and is wrong is an
error naming the file, the line and the block — a catalogue line that quietly
does nothing is the same invisible failure as a wrong angle.
