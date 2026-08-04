# A4 Maze

A single-page site that generates a complex, printable A4 maze. Press **New maze**
for a fresh one, **Print** to put it on paper. No build step, no dependencies —
one self-contained `index.html`.

## Features

- **Exact A4 output.** The sheet is 210 × 297 mm with a 12 mm margin, and
  `@page { size: A4; margin: 0 }` means the browser prints it 1:1 — no
  "shrink to fit", no clipped edges. Landscape is a toggle.
- **Five difficulties**, from a 18 × 25 grid up to 116 × 74. Cell size stays
  square and consistent between orientations.
- **Genuinely hard mazes.** A recursive-backtracker carve gives long, winding
  corridors; the harder presets are then *braided* — some dead ends are knocked
  out to create loops, which defeats the wall-following trick.
- **Reproducible.** Every maze has a seed, stored in the URL. Share the link or
  reprint later and you get the identical maze.
- **Optional solution page**, printed as page 2.

## Controls

| Action | How |
| --- | --- |
| New maze | **New maze** button, or press <kbd>R</kbd> |
| Print | **Print** button, or <kbd>Ctrl</kbd>+<kbd>P</kbd> |
| Reprint an old maze | paste its URL — the seed is in the hash |

When printing, leave margins at **Default** and scale at **100%** (uncheck
"Fit to page"). Header/footer off gives the cleanest sheet.

## Run locally

Open `index.html` in a browser. That's it — no server required.

## Deploy to GitHub Pages

```sh
git init
git add -A
git commit -m "A4 maze generator"
gh repo create maze --public --source=. --push
gh api -X POST repos/:owner/maze/pages -f build_type=legacy -f 'source[branch]=main' -f 'source[path]=/'
```

Or, without the CLI: push the repo, then **Settings → Pages → Source: Deploy
from a branch → `main` / `(root)`**. The site appears at
`https://<user>.github.io/maze/` within a minute or so.

## How it works

`index.html` holds everything. The maze lives in a `Uint8Array`, one byte per
cell, four bits marking which walls survive:

- `carve()` — iterative recursive backtracker (an explicit stack, so a
  116 × 74 grid can't blow the call stack).
- `braid()` — removes a fraction of dead ends to add loops and false trails.
- `solve()` — breadth-first search, so the drawn solution is the *shortest*
  route even when braiding created several.
- `render()` — emits one SVG `<path>` for every wall in the grid, drawn on a
  `viewBox` that scales to whatever space the sheet has left. Vector output, so
  it prints at the printer's full resolution rather than the screen's.

Randomness comes from a seeded `mulberry32` PRNG, which is what makes a seed
reproduce a maze exactly.
