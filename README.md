# Matrix Rain

> Certified drip: this repo has more falling green characters per second than your fit has ever had compliments.

A small, dependency-free browser experiment that renders animated Matrix-style rain with HTML glyphs inside an SVG `foreignObject`, then draws each frame to a low-resolution canvas and scales it up with pixelated rendering.

### I designed it to look good at 50% brightness

## Run

This requires a local web server; opening `index.html` directly from disk (`file://`) will not work correctly in most browsers due to security restrictions on the `foreignObject` rasterization technique used here. For this you can `serve` the root.

## Configuration

A few constants near the top of the `<script>` block in [`index.html`](index.html) control the animation and can be edited directly:

- `PIXEL_SIZE` — block size of the final pixelation (higher = chunkier pixels)
- `FALL_SPEED_SCALE` — higher = slower falling (multiplies frames-per-row-step)
- `STATIC_GLYPHS` — `true` keeps a trail's characters fixed as it falls, only rerolled on reset; `false` rerolls them every frame
- `chars` — the pool of characters used for the glyphs
- `ROWS_PER_TRAIL` — number of rows in each falling trail

## Real-time CSS editing

(Note: By default, .dev-column is hidden using a left -60px; if you want to see it, toggle it or put it at 0.)

The rain's glyph appearance is controlled by [`dev-column.css`](dev-column.css). Edit the `.dev-column` rule in your browser's DevTools to change properties such as font, size, color, and line height. The animation reads the rule's current computed styles and applies them to the glyph columns on every frame, so CSS changes appear in real time without restarting the page.

The visible development column is a regular DOM element, separate from the canvas, so it also provides a convenient place to inspect and experiment with the styling.

Editing `.dev-column` in the DevTools Elements/Styles panel is temporary and lost on reload. To persist your changes back to [`dev-column.css`](dev-column.css), set up a [DevTools Workspace](https://developer.chrome.com/docs/devtools/workspaces) (Sources panel → add the project folder) so edits are saved straight to the file on disk.

## What CSS works in `.dev-column`

Each frame's glyphs are rendered as plain XHTML inside an SVG `foreignObject`, which is an isolated document with no access to the host page's `<link>` stylesheets — only inline styles (read live off `.dev-column`'s computed style) get applied.

**Works:**
- `color`, `background` / `background-color`, `opacity`
- `font-family`, `font-size`, `font-weight`, `font-style`
- `line-height`, `letter-spacing`, `word-spacing`
- `text-align`, `text-indent`, `text-transform`, `text-decoration`, `text-shadow`
- `white-space` (needed as `pre` to preserve the glyph rows)
- `filter` (e.g. `blur()`, `brightness()`) — works, but adds a rasterization cost per frame

**Excluded on purpose:** `position`, `top`, `left`, `width`, `height`, `z-index` — each rain column already positions itself, so these are stripped out before being applied to the glyphs.

**Doesn't work / unreliable:**
- Anything from the host page's external stylesheet — only `.dev-column`'s own inline-read computed styles are used
- `@font-face` with relative/external URLs — often fails to load inside the isolated SVG document; stick to system fonts
- CSS animations/transitions/`:hover` — each frame is a static rasterized snapshot, not a live DOM
- `::before` / `::after` — the glyph markup is built from string concatenation, so there's no element for pseudo-content to attach to
- `vw` / `vh` / `%` — resolve against the `foreignObject`'s own box, not the real viewport

