# A4 Maze

A single-page site that makes a complex, printable A4 maze. Press **New maze** for a fresh one and **Print** to put
it on paper. No build step and no dependencies: it's one self-contained `index.html`.

**Where it lives:** <https://supersaberstacks.github.io/maze/> (GitHub Pages; not in a container).

## Using it

| Action | How |
| --- | --- |
| New maze | **New maze** button, or press <kbd>R</kbd> |
| Print | **Print** button, or <kbd>Ctrl</kbd>+<kbd>P</kbd> |
| Reprint an old maze | paste its URL: the seed is in the hash |

When printing, leave margins at **Default** and scale at **100%** (untick "Fit to page"). Turning headers and
footers off gives the cleanest sheet.

What you get:

- **Exact A4 output.** The sheet is 210 × 297 mm with a 12 mm margin, and `@page { size: A4; margin: 0 }` means
  the browser prints it 1:1: no "shrink to fit", no clipped edges. Landscape is a toggle.
- **Five difficulties**, from an 18 × 25 grid up to 116 × 74. Cells stay square and the same size in either
  orientation.
- **Genuinely hard mazes.** A recursive-backtracker carve gives long, winding corridors; the harder levels are then
  *braided*: some dead ends are knocked out to make loops, which defeats the follow-one-wall trick.
- **Reproducible.** Every maze has a seed, stored in the URL. Share the link or reprint later and you get the same
  maze.
- **An optional solution page**, printed as page 2.

## Setting it up

1. To use it on your own computer, open `index.html` in a browser. No server needed.
2. To publish it on GitHub Pages: push the repo, then **Settings → Pages → Source: Deploy from a branch →
   `main` / `(root)`**. The site appears at `https://<user>.github.io/maze/` within a minute or so. With the GitHub
   CLI, from a fresh folder:

   ```sh
   git init
   git add -A
   git commit -m "A4 maze generator"
   gh repo create maze --public --source=. --push
   gh api -X POST repos/:owner/maze/pages -f build_type=legacy -f 'source[branch]=main' -f 'source[path]=/'
   ```

## Day to day

Changes merged to `main` go live on the site within a minute or two.

## Development

`index.html` holds everything. The maze lives in a `Uint8Array`, one byte per cell, with four bits marking which
walls survive:

- `carve()`: an iterative recursive backtracker (an explicit stack, so a 116 × 74 grid can't overflow the call
  stack).
- `braid()`: removes some of the dead ends to add loops and false trails.
- `solve()`: breadth-first search, so the drawn solution is the *shortest* route even when braiding made several.
- `render()`: draws one SVG `<path>` for every wall, on a `viewBox` that scales to whatever space the sheet has
  left. It's vector output, so it prints at the printer's full resolution rather than the screen's.

Randomness comes from a seeded `mulberry32` generator, which is what lets a seed reproduce a maze exactly.
