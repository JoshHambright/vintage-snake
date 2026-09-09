# CLAUDE.md — Nightcrawler / vintage-snake

Guidance for Claude Code working in this repository.

---

## ⚠️ The VM is ephemeral. Commit and push often.

This project is developed in Claude Code cloud sessions. The container is
temporary and is reclaimed after inactivity or when the session ends. When that
happens the conversation is restored — **the filesystem is not.** Uncommitted
work is gone permanently.

**GitHub is the only durable storage.** Push at every meaningful checkpoint, not
in one batch at the end. A WIP commit that survives beats perfect work that
doesn't. Before ending a turn:

```bash
git status -sb     # tree clean? branch in sync with origin?
```

If that shows uncommitted changes or unpushed commits, you are not done.

---

## What this is

`index.html` is the whole product: Snake, played as an earthworm, on a board
that is a vertical section of garden soil drawn as a hand-coloured Victorian
engraving. One file, no dependencies, no build step, no framework. It runs by
being opened.

Live at <https://joshhambright.github.io/vintage-snake/>, deployed by
`.github/workflows/pages.yml` on every push to `main`.

If a Pages deploy fails with *"Create Pages site failed: Resource not accessible
by integration"*, Pages is not switched on for the repo. A workflow's
`GITHUB_TOKEN` cannot create the site — and `configure-pages`'s `enablement:
true` does not help, it fails the same way. Someone with repo admin has to set
**Settings → Pages → Source: GitHub Actions** once, by hand. Do not spend a
cycle trying to automate around this.

## Constraints that are already settled

Do not re-litigate these. They are the design, not defaults that happened.

- **One visual world: a printed plate.** Paper ground, sepia ink, two hand-tints.
  There is deliberately **no dark mode** — a sheet of paper does not have one. All
  colours are painted explicitly so the page holds on any host background. Do not
  add a theme toggle or `prefers-color-scheme` block.
- **No filled shapes in the artwork.** Tone is stipple density and hatching. If
  you find yourself reaching for a solid fill or a gradient to make something
  read, that is the wrong lever — change dot count, dot size, or line weight.
- **The burrow is a void left un-inked**, lighter than the ground. This inversion
  is load-bearing; it is the main thing that makes the plate read as a print. An
  earlier version drew the trail dark and it looked like a screen.
- **Exactly two hand-tints**, on the worm and the leaves. Vermilion appears once,
  in the swelled rule under the title. Adding a third colour cheapens all of them.
- **Inches, not centimetres.** Darwin measured in inches; the beds are named
  *Litter*, *Vegetable mould*, *Subsoil* — the period terms, not the modern O/A/B
  horizon codes, which are 20th-century.
- **The copy is in the register of the book** — plain-spoken, but period. Do not
  let it drift into either arcade voice ("GAME OVER! TRY AGAIN") or into
  unreadable pastiche.
- **The reference is Darwin's *The Formation of Vegetable Mould through the
  Action of Worms* (1881)**, cited in the footnote. Any new period detail should
  be true, not invented; the ten-to-eighteen-tons-per-acre figure is his.
- **No dependencies, ever.** No npm, no bundler, no library. If a change needs a
  build step, the change is wrong for this project.
- **Two colours of realism.** Anatomy is accurate on purpose (no eyes, a
  prostomium, a clitellum at ~20% of body length, castings, peristalsis). If a
  detail can be both fun and true, make it true.

## Rules that fall out of the world

Gameplay derives from the soil, not from Snake. Keep it that way — a new
mechanic should have a reason in the animal's life, not just in the genre.

- Left and right **wrap**; top (open air) and bottom (end of the section) kill.
- Leaves spawn **weighted toward the surface**, so the food sits beside the
  lethal edge. This is the entire risk curve — do not flatten the weighting.
- Eating grows the worm by three, not one.
- Every cell tunnelled stays loosened for the run.
- A pebble arrives every 5 leaves; fungal hyphæ every 4th leaf, rotting after 46
  steps.
- Tick decays 150 ms → 78 ms as leaves accumulate.

## Where things are in `index.html`

| Lines | What |
|---|---|
| 10–252 | CSS. Tokens in `:root`; the book furniture (running head, drop cap, plate frame, table, footnote) |
| 253–334 | The page markup — the "leaf" of the book |
| 340–350 | **All the tuning constants.** `COLS`, `ROWS`, bed boundaries, step timing, spawn rates |
| 373–387 | `hash()` — deterministic per-point noise, and `waver()`, a line drawn by a hand rather than a plotter |
| 389–515 | `bakeSoil()` — the engraved ground: stipple, hatching, leaf fragments, roots, bed boundaries, foxing. **Baked once per resize**, never per frame |
| 517–566 | `drawScale()` — the inch rule and the bed braces, on its own canvas |
| 635–707 | `step()` — the whole rules engine, plus the death copy |
| 709–751 | `burrowPath()` / `bakeBurrow()` — the tunnel union and its outline. Rebuilt only when a cell is added |
| 752–841 | The engraved objects: leaf, hyphæ, pebble, casting |
| 843–977 | `halfWidth()` and `drawWorm()` — the variable-width polygon, tint, clitellum, annuli hatching, contour, prostomium |
| 979–1019 | `draw()` and the frame loop |

Two offscreen canvases carry the expensive drawing: `soil` (rebuilt on resize)
and `bur` (rebuilt when `burrowDirty`). Per frame the plate is two `drawImage`
calls plus the movers. Keep it that way — anything you are tempted to draw every
frame that does not move belongs in a bake.

## The three techniques

Documented in full in the README. In short:

1. **Variable-width polygon** for the worm: taper, clitellum and peristalsis from
   one filled path.
2. **Union outline without geometry**: stroke every subpath thickly, then fill the
   same path over the top; only the outer half of each stroke survives.
3. **Wrap-safe interpolation**: adjust each segment's previous x by ±`COLS`, then
   split the body into runs where consecutive positions are >1.6 cells apart.

## Looking at your work

The design is visual, so *look at it* before pushing — but once, not in a loop.
Chromium is pre-installed in the cloud container:

```bash
cd "$SCRATCHPAD" && npm i playwright --silent
# launch with executablePath: '/opt/pw-browsers/chromium-1194/chrome-linux/chrome',
#   args: ['--no-sandbox']
```

Google Fonts is blocked by the container's proxy, so screenshots render in
fallback serifs. That is fine for judging the plate, layout and spacing; it is
not a reason to conclude the type is broken.

Write, look once, fix in one pass, push. Do not build a screenshot loop.

## What earlier passes got wrong

Recorded so they are not repeated:

1. **A vignette at 0.45 alpha** ate the lower half of the section and killed the
   grain. (From the abandoned dark version.)
2. **The clitellum pinned to fixed segment indices 6–10** lands on the *tail* of a
   short worm. It is placed proportionally now, ~20% of body length.
3. **Square burrow cells outlined at their exposed edges** engraved a machined
   duct — a floor plan, not a burrow. Hence the rounded union.
4. **Roots at one line weight running the full height** read as fissures in the
   paper. They taper now, and stop short.
5. **A taper over four of the six starting segments** made a new worm read as a
   wedge rather than an animal.

## Conventions

- Commit messages say what changed and why in the design, not just the file.
- Non-obvious visual choices get a note in the README or here — this file is the
  handoff mechanism, not decoration.
- Keep `index.html` self-contained. If it ever needs splitting, that is a
  decision to take deliberately and record here first.

## History

Began as a spike in `joshhambright/joshify` under
`spikes/earthworm-snake/`, on the branch `claude/snake-earthworm-game-rna2je`.
That copy is superseded by this repository.
