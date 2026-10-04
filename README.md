# Layered-Diagram-Generative-Art

A seed-based generative art catalogue for **Layered Diagram**, a computational linework system. Built as a single-file HTML page with p5.js, styled as a gallery-ready portfolio and licensing surface.

**Live:** `https://reyrove.github.io/Layered-Diagram-Generative-Art/`

---

## Overview

**Layered Diagram** renders three distinct layers of linework from numeric seeds. Each composition stacks a regular grid, 6–8 organic random-walk curves, and 3–5 sets of concentric circles — all drawn in black on white, without hierarchy or fill. The result reads as a drawing in three registers at once: measurement, motion, and structure.

The page functions as a **catalogue and licensing surface** for fashion houses, textile studios, and surface designers. Every composition is reproducible by seed, meaning the same number always produces the identical diagram.

---

## Features

### Generative Engine (p5.js)

- **Layer 1 — Regular grid** — vertical and horizontal lines at 2.5% of canvas spacing, extending to a 5% margin from each edge
- **Layer 2 — Organic curves** — 6–8 random walks, each 100 steps long, drifting across the canvas in smooth angular steps (angle delta ±0.5 rad, radius 0.3–0.8% of canvas)
- **Layer 3 — Concentric circles** — 3–5 sets, each centred at a random point and containing 3–5 rings at increasing radii with slight radial variance (±0.15% of canvas)
- **Heavy border** — a double-weight frame at 5% margin (`cs × 0.002`)
- **Thin lines** — interior linework at `cs × 0.001`
- **Black on white** — no fills, no colours, no stroke variance between layers

### Same Seed → Many Surfaces

A single seed drives every instance on the page:

| Instance | Purpose | Tiling |
|----------|---------|--------|
| **Cover** | Hero composition | 1 × 1 |
| **Plate** | Main work (master) | 1 × 1 |
| **Frame** | Framed archival print | 1 × 1 |
| **Print** | Archival print surface | 1 × 1 |
| **Scarf** | Silk scarf (3:1) | 3 × 1 |
| **Textile** | Fabric yardage (4:3) | 2 × 2 |
| **Wall** | Wallpaper (2:3) | 1 × 2 |
| **Archive** | 8 unique seeds | 1 × 1 each |

### Controls

- **New Seed** button / `R` key — regenerate with a fresh seed
- **Download** button / `S` key — export the plate canvas as PNG
- **Click any archive plate** — load that seed into the main view

### Performance

- Debounced resize handling (250ms)
- `IntersectionObserver` pauses off-screen sketches
- `visibilitychange` pauses rendering when the tab is hidden
- `pixelDensity(1)` for a lighter canvas footprint
- Two-frame render (`maxFrames: 2`) so tiles draw once and stop

---

## File Structure

```
Layered-Diagram-Generative-Art/
├── index.html          # Single-file page (structure + styles + generative engine)
├── README.md
└── images/
    ├── fav.svg         # Favicon
    ├── tote.png        # Mockup: tote bag
    ├── tee.png         # Mockup: t-shirt
    └── cushion.png     # Mockup: cushion
```

No build step. No dependencies to install. p5.js is loaded from a CDN.

---

## Getting Started

1. Clone the repo:
   ```bash
   git clone https://github.com/reyrove/Layered-Diagram-Generative-Art.git
   cd Layered-Diagram-Generative-Art
   ```
2. Add the three mockup images to `images/` (optional — they appear in the Commission section).
3. Open `index.html` in a browser.

That's it. The page self-initialises on `DOMContentLoaded`.

---

## How the Generative System Works

### Seeded Random

A linear congruential generator replaces p5's native randomness inside each sketch:

```js
class SeededRandom {
  constructor(s) { this.state = (Math.abs(s) % 2147483647) || 1; }
  rand() { this.state = (this.state * 9301 + 49297) % 233280; return this.state / 233280; }
  range(min, max) { return this.rand() * (max - min) + min; }
  int(min, max) { return Math.floor(this.range(min, max + 1)); }
  pick(arr) { return arr[Math.floor(this.rand() * arr.length)]; }
}
```

Every aesthetic decision — curve count, curve walk, circle count, circle centres, radial variance — is drawn from this generator in a fixed order. Same seed, same sequence, same composition.

### Parameter Space (per seed)

| Parameter | Range |
|-----------|-------|
| Canvas margin | 5% of canvas |
| Grid spacing | 2.5% of canvas |
| Curve count | 6 – 8 |
| Curve steps | 100 |
| Curve angle delta | ±0.5 rad per step |
| Curve radius | 0.3% – 0.8% of canvas |
| Circle set count | 3 – 5 |
| Rings per set | 3 – 5 |
| Ring radial variance | ±0.15% of canvas |
| Ring steps | 60 |

### The Three Layers

The three layers are drawn in sequence, one after the other, on the same canvas — no masking, no blending, no transparency. The grid draws first (bottom), then the curves, then the circles (top). Because everything is black linework, the eye reads the composition as a single drawing rather than three superimposed drawings.

**Grid:**
```js
for (let x = margin; x <= cs - margin; x += gridSpacing) {
  p.line(x, margin, x, cs - margin);
}
for (let y = margin; y <= cs - margin; y += gridSpacing) {
  p.line(margin, y, cs - margin, y);
}
```

**Curves (random walk):**
```js
let currentAngle = sr.rand() * Math.PI * 2;
for (let t = 0; t < 100; t++) {
  currentAngle += sr.range(-0.5, 0.5);
  const radius = sr.range(cs * 0.003, cs * 0.008);
  // advance walker; clamp to margin
}
```

**Circles (concentric with variance):**
```js
for (let j = 0; j < concentricCount; j++) {
  const r = baseRadius * (j + 1) * sr.range(0.8, 1.2);
  for (let a = 0; a <= TWO_PI; a += TWO_PI / 60) {
    const variance = sr.range(-cs * 0.0015, cs * 0.0015);
    // draw circle point at (r + variance)
  }
}
```

### Tiling Logic

Rectangular surfaces (scarf, textile, wall) tile the square composition N×M. Each tile is drawn into its own translated coordinate space, centred within its cell:

```js
const tileW = canvasW / tileCols;
const tileH = canvasH / tileRows;
const tileSize = Math.min(tileW, tileH);
// draw square of side tileSize at (ox, oy) for each cell
```

This preserves the square composition's proportions regardless of surface aspect ratio.

---

## Customisation

### Change the default seed

Edit the init function:

```js
function init() {
  seed = 123456789;  // fixed seed instead of random
  // ...
}
```

### Adjust the grid resolution

```js
const gridSpacing = cs * 0.025;  // change 0.025 for finer or coarser grid
```

Smaller values (e.g., `0.01`) produce a dense drafting-paper feel; larger values (e.g., `0.05`) produce a sparse, structural grid.

### Adjust the curve behaviour

Inside the curve layer:

- **Curve count:** `sr.int(6, 8)` — change for more or fewer walks.
- **Walk length:** `t < 100` — change for shorter or longer curves.
- **Angle delta:** `sr.range(-0.5, 0.5)` — smaller values produce smoother, more directional walks; larger values produce wilder, more erratic paths.
- **Radius:** `sr.range(cs * 0.003, cs * 0.008)` — smaller values produce finer, more detailed walks; larger values produce broader, sweeping curves.

### Adjust the circle behaviour

- **Set count:** `sr.int(3, 5)` — number of concentric groups.
- **Rings per set:** `sr.int(3, 5)` — number of rings in each group.
- **Radial variance:** `sr.range(-cs * 0.0015, cs * 0.0015)` — larger values produce wobblier circles; zero produces perfectly circular rings.

### Change the stroke weight

```js
p.strokeWeight(cs * 0.001);   // interior linework
p.strokeWeight(cs * 0.002);   // border
```

Increase for a heavier ink, decrease for a hairline. The ratio between interior and border is currently 1:2 — change either to taste.

### Change the ground

```js
p.background(255);
```

Replace with a paper tone for a warmer ground:

```js
p.background(245, 240, 230);   // cream
```

Or invert the composition entirely:

```js
p.background(0);
p.stroke(255);
```

### Change surface tiling

Edit `surfaceDefs`:

```js
const surfaceDefs = [
  { id: 'surfacePrint',   wrapId: 'wrap-surfacePrint',   tile: { cols: 1, rows: 1 } },
  { id: 'surfaceScarf',   wrapId: 'wrap-surfaceScarf',   tile: { cols: 3, rows: 1 } },
  // ...
];
```

The `tile` object accepts any `{ cols, rows }` pair.

### Replace mockup images

Swap the files in `images/` or update the `src` attributes in the Commission section:

```html
<img src="images/your-mockup.png" alt="..." loading="lazy" />
```

---

## Design System

| Token | Value | Use |
|-------|-------|-----|
| `--bg` | `#0a0a0a` | Page background |
| `--paper` | `#f4f1ea` | Light surface (Surfaces section) |
| `--gold` | `#b8935a` | Primary accent |
| `--gold-soft` | `#d4b483` | Italic emphasis |
| `--serif` | Cormorant Garamond | Headings, titles |
| `--mono` | DM Mono | Labels, metadata, UI |

Type scale, spacing, and section rhythm match the Skyline, Portrait, Girih, Gothic Grid, Order to Chaos, Wallpaper Groups, Wandering Tiles, Food Grid, Framed Line, and Deconstructed Stage templates. Responsive breakpoints at 900px, 720px, 560px, and 400px.

---

## Keyboard Shortcuts

| Key | Action |
|-----|--------|
| `R` | Regenerate with a new seed |
| `S` | Download plate as PNG |
| `Enter` / `Space` | Activate focused archive item |
| `Esc` | Close legal modal |

---

## Browser Support

Modern evergreen browsers with:

- Canvas 2D (universal)
- `IntersectionObserver` (universal in modern browsers)
- `backdrop-filter` (Safari 9+, Chrome 76+, Firefox 103+)
- CSS custom properties (universal)

Fallback: if `IntersectionObserver` is unavailable, sketches render continuously.

---

## Accessibility

- Archive items are keyboard-focusable with `role="button"` and `aria-label`
- Legal modal traps focus, closes on `Esc` and overlay click
- `prefers-reduced-motion` disables transitions and smooth scroll
- All interactive elements have visible focus states (via browser default)

---

## Section Map

| Section | ID | Purpose |
|---------|-----|---------|
| Cover | — | Hero title + live diagram |
| Statement | `#statement` | Artist statement |
| Plate | `#work` | Main work + metadata + controls |
| Surfaces | `#application` | Framed print + 4 tiled surfaces |
| Process | `#process` | Three process cards |
| Archive | `#archive` | 8 selectable seeds |
| Commission | `#commission` | Services + mockups + CTA |
| Colophon | — | Footer, links, legal |
| Legal modal | `#legalModal` | Licensing / Terms / Credits |

---

## Deployment (GitHub Pages)

1. Push the repo to GitHub.
2. Go to **Settings → Pages**.
3. Under **Source**, select `Deploy from a branch`.
4. Choose branch `main` and folder `/ (root)`.
5. Save — the site goes live at `https://reyrove.github.io/Layered-Diagram-Generative-Art/`.

---

## Notes on the Source Sketch

The generative engine is adapted almost verbatim from the original p5 sketch. The adaptations that turn it into a catalogue page:

1. **Seeded random replaces p5's `random()`** — a local `SeededRandom` class drives every choice. The original sketch calls `setSeeds()` for fxhash compatibility; that's replaced with the seeded generator directly.
2. **Sketch factory** — `makeSketch({ slotId, seedValue, ... })` returns a p5 sketch that can be instantiated into any DOM slot, enabling cover / plate / frame / surfaces / archive from one seed.
3. **DOM-driven canvas sizing** — `cs = min(windowHeight, windowWidth)` becomes slot-rect dimensions.
4. **Tiling support** — the `tile: { cols, rows }` parameter repeats the square composition across rectangular surfaces.
5. **Drawing logic takes `p` explicitly** — the original used p5's global mode; here `p` is passed so the factory closure works.
6. **`constrain(...)` inlined with `Math.max` / `Math.min`** — so the drawing code doesn't depend on p5 globals inside the factory.
7. **`windowResized` replaced by a debounced `handleResize`** — all sketches are destroyed and recreated on resize (250ms debounce), since their canvases are locked to slot dimensions.
8. **`triggerPreview` removed** — the original calls it for fxhash thumbnail generation; not needed in the plain-browser catalogue.

---

## Related Repositories

| Repo | Theme |
|------|-------|
| [`Skyline-Series`](https://github.com/reyrove/Skyline-Series) | Themed cityscapes |
| [`Portrait-Series-Generative-Art`](https://github.com/reyrove/Portrait-Series-Generative-Art) | Generative faces |
| [`Girih-2-Generative-Art`](https://github.com/reyrove/Girih-2-Generative-Art) | Islamic geometric ornament |
| [`Gothic-Grid-Generative-Art`](https://github.com/reyrove/Gothic-Grid-Generative-Art) | Mosaic grid composition |
| [`Order-to-Chaos-Generative-Art`](https://github.com/reyrove/Order-to-Chaos-Generative-Art) | Gradient grid composition |
| [`Wallpaper-Groups-Generative-Art`](https://github.com/reyrove/Wallpaper-Groups-Generative-Art) | Crystallographic symmetry |
| [`Wandering-Tiles-Generative-Art`](https://github.com/reyrove/Wandering-Tiles-Generative-Art) | Line-drawing composition |
| [`Food-Grid-Generative-Art`](https://github.com/reyrove/Food-Grid-Generative-Art) | Illustrated food grid |
| [`Framed-Line-Generative-Art`](https://github.com/reyrove/Framed-Line-Generative-Art) | Gesture line composition |
| [`Deconstructed-Stage-Generative-Art`](https://github.com/reyrove/Deconstructed-Stage-Generative-Art) | Suprematist composition |
| **`Layered-Diagram-Generative-Art`** | Layered linework composition *(this repo)* |

All eleven share the same catalogue template, design tokens, and seed-based reproducibility model.

---

## Credits

- **Design & Generative System** — Reyhaneh Daneshdoost
- **Typefaces** — Cormorant Garamond, DM Mono
- **Engine** — p5.js 1.9.0
- **Edition** — Layered Diagram, Autumn 2026

---

## License

All artwork, code, and generative systems are the intellectual property of the artist. Compositions may not be reproduced, redistributed, resold, or adapted without a written license.

For licensing, commissions, or private systems: **reyhanehdaneshdoost@gmail.com**