# circle

A circle drawn as four cubic Bézier segments, for
[Vibrant Gio](https://github.com/vibrantgio), a design system for native desktop
applications on macOS, Windows and Linux, written in pure Go on
[Gio](https://gioui.org). Three functions, no state.

Gio ships `clip.Ellipse`, and it takes an `image.Rectangle` — integer pixels,
axis-aligned, sized by its bounds. That is exactly right for a rounded avatar
in a layout and exactly wrong for a particle: a physics simulation produces a
`float32` centre and a `float32` radius many times a second, and rounding both
to a pixel grid every frame makes small circles jitter as they move and makes a
scaled scene snap between sizes.

`CirclePath` takes an `f32.Point` centre and a `float32` radius and returns a
`clip.PathSpec`, so the geometry stays in floating point all the way to the
rasteriser. It is the reason
[traer](https://github.com/vibrantgio/traer)'s particle demos render smoothly
under continuous zoom.

The Bézier constant is the other reason this module exists. Approximating a
quarter circle with a cubic needs a control-point offset of *c·r*, and the
number everyone quotes is 0.55228475 — `4·(√2−1)/3`, derived by making the
curve pass exactly through the quarter point. That choice leaves the maximum
radial error where it is largest. This module uses Spencer Mortensen's
0.5519150244935105707435627, which minimises the maximum error instead, and is
the better constant for a circle you will scale.

## Where it sits

Tier 0 of the stack — `mvu → theme → components → effects → cadence → markdown` —
a leaf whose only dependency is Gio. The
[organization page](https://github.com/vibrantgio) has the full tier table.

It sits alongside [backdrop](https://github.com/vibrantgio/backdrop) and
[gradient](https://github.com/vibrantgio/gradient), the other two drawing
leaves. Nothing inside the design system imports it — components draws its own
shapes — and its consumers are demos: `mvu/example/circles`, and all four of
[traer](https://github.com/vibrantgio/traer)'s `gio` particle programs.

```sh
go get github.com/vibrantgio/circle
```

Every module in the organization is on gioui.org v0.10.1 and Go 1.25.1.

## Packages

One package, at the module root.

| Symbol | |
| --- | --- |
| `CirclePath(ops, p, r)` | The `clip.PathSpec`: four cubics around centre `p` with radius `r`, closed. Use it with `clip.Outline` — the path is a single closed contour and relies on non-zero winding. |
| `FillCircle(ops, p, r, fill)` | `CirclePath` clipped and painted in one call. Takes `color.Color` and converts through `color.NRGBAModel`. |
| `Circle(p, r, fill)` | `FillCircle` as a `layout.Widget`. Returns `gtx.Constraints.Max` — see Status. |

## Usage

The path is the interesting one, because it composes. This is
`traer/gio/arboretum`, drawing the root node of a spring-laid-out tree: the
particle's position and the current zoom are both floats, and neither is
rounded on the way to the rasteriser.

```go
particle := ps.Particles[0]
p := f32.Point{X: float32(particle.Position.X), Y: float32(particle.Position.Y)}
cstack := clip.Outline{Path: circle.CirclePath(ops, to(p), 3*nodesize*scale)}.Op().Push(ops)
paint.ColorOp{Color: DeepPurple800}.Add(ops)
paint.PaintOp{}.Add(ops)
cstack.Pop()
```

Pushing the clip yourself is what buys you the composition — anything painted
between the push and the pop is masked by the circle, so the same call fills
with a solid, a gradient or an image.

When you just want a disc, `FillCircle` is the whole thing. From
`mvu/example/circles`, which grows a circle under the pointer while it is
dragged:

```go
for _, c := range circles {
	vcircle.FillCircle(gtx.Ops, c.Center, c.Radius, colornames.Yellow800)
}
```

`c.Center` there comes straight off `pointer.Event.Position`, a `f32.Point` —
no rounding, and the circle tracks the cursor sub-pixel.

## For coding assistants

Read the canonical guide before writing code against this module — the module
inventory with current tags, the application skeleton, MVU and rx semantics,
typography, and the pitfalls that are not guessable:

<https://raw.githubusercontent.com/vibrantgio/.github/master/llms.txt>

[`AGENTS.md`](./AGENTS.md) in this repository has the build and test commands.

## Status

Honest about what does not work yet. Every count below is measured.

- **`Circle`, the widget, has no consumer anywhere in the organization.** Every
  one of the five call sites uses `CirclePath` (three) or `FillCircle` (two);
  the `layout.Widget` form has never been called. It is also the one form with
  a design problem: it returns `layout.Dimensions{Size: gtx.Constraints.Max}`
  whatever the radius is, and its centre `p` is an absolute point in the
  widget's own coordinate space rather than a position within the space it was
  given. So in a flex it claims all the room and draws wherever it was told,
  which is not what a widget is for. Use `FillCircle` inside a widget you wrote.
- **Gio can already do this, less precisely.** `clip.Ellipse` is an
  `image.Rectangle` and gives you an axis-aligned ellipse inscribed in it. For
  a static circle in a layout that is the simpler call and this module adds
  nothing; the float32 centre and radius are the whole reason to reach for
  `CirclePath` instead. Nothing in the repository says so, and there is no
  benchmark comparing them.
- **Circles only — no arcs, no ellipses, no strokes.** The four cubics are
  hard-coded as a closed loop, so there is no way to draw three quarters of a
  circle, an ellipse with different radii, or an outline of a given width. A
  ring today is two filled circles, the inner one in the background colour.
- **Nothing in the design system uses it.** components, effects, cadence, markdown and
  theme all draw their own shapes; the consumers are `mvu/example/circles`
  and the four `traer/gio` demos. No component in the stack renders a circle
  through this module, so its behaviour under the theme, at high DPI, and
  against the golden-image tests the rest of the stack has is entirely
  unexercised.
- **The Bézier claim is not tested.** The constant is right and the comment
  explaining it is right, but nothing measures the resulting radial error, and
  `go test ./...` reports "no test files". The precision this module exists for
  is asserted, not pinned.
- **There is no LICENSE file.** Eighteen of the organization's twenty
  repositories ship one — MIT for most, BSD, Apache or Unlicense for the ported
  libraries. This one and [gradient](https://github.com/vibrantgio/gradient)
  are the two that do not, and no phase of the current plan adds them.
