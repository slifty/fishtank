# AGENTS.md

Notes for AI agents (and humans) working on this project.

## The shape of the project

This is a **single-file** app. Everything — HTML, CSS, and all JavaScript —
lives in **`index.html`**. There is **no build step, no bundler, no
dependencies, and no package.json**. You edit `index.html` and reload the page.

```
index.html                  ← the entire app (markup + CSS + JS)
README.md                   ← player-facing overview
AGENTS.md                   ← this file
LICENSE
.github/workflows/deploy.yml ← publishes the repo root to GitHub Pages on push to main
```

## Running it

Open `index.html` in a browser. To preview locally over HTTP you can run any
static server, e.g. `python3 -m http.server` and open the printed URL. There
are no tests and nothing to compile — verification is visual, in the browser.

## How `index.html` is organised

Reading top to bottom, the file is laid out as:

1. **`<style>`** — all CSS. Fullscreen `#tank` canvas, a fixed bottom
   `#toolbar` of round `.btn` buttons, a `#count` readout, the audio `#hint`,
   and the `#changelog` modal. Note the screensaver behaviour: `body.idle`
   fades the toolbar and counter out.
2. **`<body>`** — the `<canvas id="tank">`, the toolbar buttons (each with an
   `id` like `bFood`, `bFish`, `bDay`…), and the `#changelog` panel markup.
3. **`<script>`** — the simulation. Roughly in order:
   - Canvas/context setup + `resize()` (handles devicePixelRatio).
   - Small helpers: `rand`, `pick`, `clamp`, `lerp`, `dist`, `hsl`, `mixHex`,
     `TAU`, the `NAMES` list.
   - `Sound` — an IIFE wrapping the WebAudio bits (starts muted/locked).
   - **Genetics**: `randomGenes()`, `breed()`, `BODY_SHAPES`/`TAILS`/`PATTERNS`.
     Fish traits (glow, bubbles, horn, whiskers, eyelashes, …) live in genes.
   - **Entity arrays**: `fishes`, `crabs`, `seahorses`, `foods`, `bubbles`,
     `hearts`, `decorations`, `castles`, plus `fishById` and `nextId`.
   - **Classes**, each with `update(dt)` and `draw()`: `Fish` (by far the
     biggest), `Food`, `Crab`, `Seahorse`, `Turtle`, `Bubble`, `Heart`,
     `Plant`, `Rock`, `Castle`, `Chest`, and transient `visitors` (`Whale`,
     `Jellyfish`, `Penguin`) triggered by `triggerSurprise()`.
   - **Relationships** (friends/enemies) are `Set`s of entity `id`s stored on
     each fish *and* on the single `turtle`. `fishById` maps every id (fish +
     turtle) to its object, so highlight/selection code can resolve them. The
     turtle has `parents: []` so `bloodline()`/`related()` treat it as having
     no family (it never breeds). `selected` may be a fish or the turtle;
     `drawHoverHighlights()` iterates `fishes.concat(turtle)` so a relationship
     shows whichever side is tapped.
   - **Fish death**: a fish sets `dead` (and `starved`) on itself; the frame
     loop's removal pass calls `removeFishAt()`, which scrubs its id from every
     friends/enemies set and `fishById`. Starved fish burst into `Food`; fish
     eaten by the turtle just vanish.
   - **Day/night**: `tankClock`, `daylight`, `dayMode` (`"cycle" | "day" |
     "night"`), `DAY_PERIOD`, `drawBackground()`, `drawNightVeil()`.
   - **Interaction/hit-testing**: `tankXY(e)` maps a pointer event to tank
     coords; `fishAt(x, y)` returns the fish under a point; `selected` holds
     the tapped fish; `drawHoverHighlights()` draws the family/friends/enemies
     overlay and info for the selected fish.
   - **Main loop**: `frame(now)` — computes `dt`, runs every entity's
     `update(dt)`, then draws everything in z-order, then
     `requestAnimationFrame(frame)`. This is the heartbeat; see the draw-order
     section below.
   - **Event wiring** at the bottom: a `pointerdown` on the canvas does the
     tap-a-fish / drop-food logic; each toolbar button has a `click` handler;
     idle/wake handlers drive the screensaver fade and unlock audio.

## Key conventions and gotchas

- **Everything visible is canvas paint, not DOM.** The night dimming, info
  overlay, rings, etc. are drawn with `ctx`, not HTML elements. The only real
  DOM overlays are the toolbar, the counter, the hint, and the `#changelog`
  modal. If you're chasing a "clicks don't register" bug, remember the canvas
  has a single `pointerdown` listener — a canvas region can't be "blocked"
  except by a real DOM element sitting on top with `pointer-events: auto`.
- **`pointer-events` care with modals.** A child with an explicit
  `pointer-events: auto` is *not* disabled by a parent's `pointer-events:
  none`. The `#changelog .card` had exactly this bug — when hidden it kept
  eating clicks in the centre of the tank. When hiding an overlay, make sure
  the interactive children are disabled too (see the `#changelog.hidden` rule).
- **Hit radius vs. visual size.** `fishAt` uses `d < f.len * 0.7`, but glowing
  fish render a halo well beyond that (especially at night), so a fish can look
  bigger than its clickable area. Fish with `f.inside` (hidden in a castle) are
  intentionally not selectable.
- **Draw order = z-order.** Things drawn later in `frame()` sit on top. If
  something should appear in front of/behind fish, move its draw call relative
  to `fishes.forEach(f => f.draw())`.
- **`dt` is clamped** to 0.05s max so a backgrounded tab doesn't fast-forward
  the sim. Animate using `dt`, not frame counts.
- **DPR-aware canvas.** `resize()` scales for `devicePixelRatio`; work in CSS
  pixels (`W`/`H`), which already account for it.
- **IDs are the wiring.** Buttons connect by `id` (`bFood`, `bFish`, `bCrab`,
  `bHorse`, `bPlant`, `bDay`, `bLog`, `bSound`). Adding a control means adding
  both the markup and a handler at the bottom of the script.

## Changelog

There's an in-app "what's new" panel (`#changelog`, opened with the 📜 button).

**Update it whenever you make an improvement to the game** — a new feature,
creature, trait, or other player-visible enhancement gets an entry so players
see what's new. **Do not** add entries for ordinary bug fixes; only note a fix
here if it's major (a significant, clearly player-visible change in behaviour).
When in doubt: improvements yes, routine fixes no.

## Deploying

Merging to `main` triggers `.github/workflows/deploy.yml`, which publishes the
repo root to GitHub Pages. No build, no artifacts — the live site is just
`index.html` served statically.
