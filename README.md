# Nightcrawler

**[▸ Play it](https://joshhambright.github.io/vintage-snake/)**

Snake, but the snake is an earthworm — and the board is a vertical section of
garden soil, drawn as a hand-coloured plate from a Victorian natural-history
text.

One HTML file. No dependencies, no build step, no network calls except the
Google Fonts stylesheet. Open `index.html` in any browser and it runs.

---

## The premise

A reskin of Snake is a worm-shaped snake. The premise here is that the *board*
becomes the animal's world, and the rules fall out of the soil rather than
being inherited from the arcade:

| Snake | Nightcrawler | Why |
|---|---|---|
| Walls kill on all four sides | Left and right **wrap**; top and bottom kill | The mould continues at either hand. Above is open air; below, the subsoil ends. |
| Food spawns uniformly | Leaf litter is **weighted toward the surface** | Leaf-fall really does collect in the top inches — so the food sits beside the lethal edge. That is the whole risk curve. |
| Grows by one | Grows by **three** | Worms get long, not fat. |
| The trail is invisible | Every inch tunnelled stays **loosened**, permanently | The burrow network is the run's record — and aeration is what the animal is actually for. |
| — | A **pebble** works into the section every 5 leaves | Escalation that belongs to soil, not to a cabinet. |
| — | **Fungal hyphæ** appear every 4th leaf and rot after 46 steps | A timed bonus with a reason to exist. |

Death copy is written in the register of the book: surfacing gets you a thrush,
the bottom edge is where the section ends at twenty inches, and self-collision
observes that a worm has no eyes whatever and maps the ground by touch alone.

## The look

The reference is the actual Victorian literature on the subject — Darwin's *The
Formation of Vegetable Mould through the Action of Worms* (John Murray, 1881),
his last book, which is about earthworms and the soil they make. So the page is
set as a leaf from that kind of book: running head and folio, drop cap,
double-ruled plate with an italic *Fig. 1* caption, a ruled table of
observations, a footnote.

What "engraved" means in the drawing:

- **No filled shapes.** Tone comes from stipple density and angled hatching, the
  way a plate was darkened before halftone. The three beds differ by dot count
  and size — 6 coarse, 15 medium, 27 fine per cell — and the subsoil takes a
  second set of lines that crowd with depth.
- **The burrow is a void the engraver leaves un-inked.** It reads *lighter* than
  the ground, not darker. That single inversion is what makes the plate read as
  a print rather than a screen.
- **Two hand-tints only** — pale flesh on the worm, sage on the leaves — laid
  inside the line the way a colourist worked. Everything else is sepia on paper.
- **The scale is in inches**, because Darwin measured in inches: twenty rows,
  twenty inches, with the beds named *Litter*, *Vegetable mould* and *Subsoil*,
  the period terms rather than the modern O/A/B horizons.
- Vermilion appears exactly once, in the swelled rule beneath the title.

Type is Libre Caslon Display over Old Standard TT — the latter designed to
reproduce late-19th-century scientific book setting.

## Controls

| | |
|---|---|
| Arrow keys / `W A S D` | steer |
| `Space` | suspend the observation |
| `R` | begin afresh |
| Swipe | steer, on a touch-screen |

## Three techniques worth stealing

**The worm is one filled polygon of variable half-width**, not a chain of
circles or a fixed-width stroke. Per segment, take the local tangent from
`pos[i+1] - pos[i-1]`, get its normal, and emit a left/right pair offset by a
half-width that varies:

```js
let w = cell * 0.33;
if (fromTail < 3) w *= 0.56 + fromTail * 0.155;            // taper
if (i >= clitStart(n) && i < clitStart(n) + 3) w *= 1.20;  // clitellum
w *= 1 + 0.13 * Math.sin(phase - i * 0.62);                // peristalsis
```

Fill `left[0..n]` then `right[n..0]` as one closed path. That buys the taper,
the clitellum swelling, and a peristaltic wave travelling head-to-tail — which
is how the animal actually moves — from a single shape.

**Outlining a union without a geometry library.** The burrow is overlapping
circles bridged to their neighbours. To get one clean outline around the whole
network, build the path twice: stroke every subpath at 2.3px, then fill the same
path with paper on top. The fill covers every interior stroke and leaves only
the outer half of each. Nine lines, no boolean ops, and it works for any blobby
union on a 2D canvas.

**Wrap-safe interpolation.** Rendering lerps between `prevWorm` and `worm` each
step so a 78–150 ms tick glides. At the wrap seam, adjust each segment's
previous x by ±`COLS` so it slides *off* the edge, then split the body into runs
wherever consecutive rendered positions are more than 1.6 cells apart, and draw
each run separately. Without the split you get a stripe across the plate.

## Deployment

Pushing to `main` runs `.github/workflows/pages.yml`, which uploads the repo
root and deploys it to <https://joshhambright.github.io/vintage-snake/>. There
is no build step to break.
