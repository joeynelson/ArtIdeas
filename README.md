# ArtIdeas

Small, self-contained generative art pages. No build step, no dependencies — open the
HTML file in a browser (or serve the folder) and it runs.

## Spiral Voronoi

[`voronoi-spiral/index.html`](voronoi-spiral/index.html)

A Voronoi mosaic grown from a spiral of seed points. The spiral slowly rotates,
breathes, and shears, so cells stretch and swap neighbours and moiré arms drift
through the tiling. Colour is sampled from cyclic [Oklab](https://bottosson.github.io/posts/oklab/)
ramps by radius and angle, which is what makes the bands read as spirals rather
than rings.

Everything is computed on a plain 2D canvas:

- **Exact Voronoi, no library.** Each cell is the frame rectangle clipped by the
  perpendicular bisector against nearby seeds (Sutherland–Hodgman half-plane
  clipping). A uniform grid supplies the candidates, and expansion stops once
  `k * cellSize >= 2R` — a seed at distance `d` can only cut points farther than
  `d/2` away, so beyond that ring the cell is final.
- **Perceptual colour.** Palettes are interpolated in Oklab and baked into a
  1024-entry lookup table at startup, so per-frame colouring is a table read.
- **One drawing pass.** Cells are filled and rimmed in the same loop on
  `source-over`; switching `globalCompositeOperation` mid-frame turned out to
  cost more than all the geometry and rasterisation combined. The film grain is
  a CSS `mix-blend-mode` layer above the canvas for the same reason — it is
  composited once instead of being re-filled 60 times a second, and gets folded
  back into the bitmap when you save a PNG.

Default settings hold 60 fps in a software rasteriser; the panel shows a live
frame rate if you push the point count up.

### Controls

Press <kbd>H</kbd> to show/hide the panel.

| | |
|---|---|
| **spiral** | `phyllotaxis` (golden-angle sunflower), `archimedean` (evenly spaced arms), `logarithmic` (nautilus, dense at the centre) |
| **points** | 48–2400 seeds (about 60% of them land on screen; the disc has to overshoot the corners) |
| **turns / arms** | winding and number of interleaved arms (Archimedean and logarithmic only) |
| **twist** | shear amplitude — outer cells rotate against inner ones. It sways sinusoidally through zero, so the spiral winds one way, unwinds to an untwisted state, then winds the other way |
| **sway** | how long one full twist cycle takes at speed 1 (150 mHz ≈ 7s up to 2 mHz ≈ 500s). Changing it alters the rate without jumping the current position, so the motion stays continuous |
| **swirl / rings** | angular and radial frequency of the colour ramp |
| **speed** | 0 freezes the piece as a still |
| **grout** | insets each cell to open gaps between them |
| **rims / seeds / grain / cursor** | edge highlight, seed dots, film grain, pointer repulsion |

<kbd>space</kbd> pause · <kbd>R</kbd> shuffle · <kbd>S</kbd> save PNG · <kbd>F</kbd> fullscreen · <kbd>1</kbd>–<kbd>7</kbd> palette

Honours `prefers-reduced-motion` by starting with speed at 0. `window.SpiralVoronoi`
exposes `params`, `set`, `pause`, `shuffle`, `setPalette`, `save`, and `snapshot()`
if you want to drive it from the console.
