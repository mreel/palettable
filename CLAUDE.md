# Palettable

An OKLCH color palette picker. The entire app is a single self-contained
`index.html` — open it in a browser, no build step, no server required.

## Core principle: NO DEPENDENCIES

This project is intentionally dependency-free and must stay that way.

- **No npm packages, no `package.json`, no `node_modules`.**
- **No build tools, bundlers, or transpilers.**
- **No CDN links, external scripts, stylesheets, or fonts.** Everything ships
  inside `index.html` (inline `<style>` and `<script>`).
- **No frameworks** — vanilla HTML/CSS/JS only.

If a feature seems to need a library, write it by hand instead.

## Running and verifying

**A dev server does not work from this folder.** The project lives under a
OneDrive path where `python3 -m http.server` (and anything else that resolves
the working directory through pyenv) fails with `getcwd: Operation not
permitted`. Don't burn time trying to start one.

To verify changes:

- **Visual:** open `index.html` directly in a browser (`file://`). It is a
  static single file, so this exercises everything.
- **Logic:** extract the inline script and run it under a small DOM shim in
  node. This catches runtime errors in event handlers and the initial render,
  and lets the color math be checked numerically (e.g. `Cmax` against a
  brute-force gamut search). This is how the color work in this repo was
  validated.

## Architecture

All of it is inside the one `<script>` in `index.html`.

- **Color math** — sRGB encode/decode, OKLab ↔ linear RGB, OKLCH ↔ OKLab.
- **`COLOR_SPACES`** — registry of `srgb` / `displayP3` / `rec2020`. Each entry
  holds the OKLab-LMS → linear-RGB matrix (its rows double as the gamut
  solver's per-channel weights) plus the CSS `color()` id. Adding a gamut is
  one more entry.
- **`activeSpace`** — defaults to `'srgb'` **on purpose**. Wide gamuts are
  wired up and selectable, but sRGB is the current target.
- **Gamut solver** — `Cmax(H, L)` is exact and closed-form: at fixed L,H each
  channel is a cubic in C, so the boundary is the smallest positive root where
  a channel reaches 0 or 1 (Cardano). Space-agnostic — same math every gamut.
  It replaced an earlier binary search; don't reintroduce one.
- **Generators** — `genLightness` / `genHue` / `genChroma` build the palette
  from the key color.
- **Rendering** — `renderAll()` fans out to controls, rails, key readout, and
  palette. Slider rails are live gradients painted as the slider `background`.

### Two picker modes

`pickerMode` is either `'sliders'` (default, the three Hue/Lightness/Chroma
sliders) or `'square'` (a 2D canvas picker where the active tab decides which
axis is the slider and which two fill the square). `setPickerMode(mode)` is the
only seam between them; a view-toggle button calls it.

The square-mode code (`MODE_CFG`, axis helpers, `renderSquareBg`,
`positionMarker`, `configureSlider`, the drag/keyboard handlers) is fully
functional but skipped entirely while in slider mode. **The square's UI
placement is not settled** — the current toggle button is a placeholder.

### Lightness generator: intended behavior

`genLightness` spans the **full 0–1 lightness axis** as a rigid window centred
on the key lightness, shifted (never clamped per-endpoint) to fit, with each
step gamut-mapped. Consequences, all deliberate:

- Range 100 runs **pure black to pure white**.
- Chroma is **held at the key value** and only pulled down where the gamut
  forces it, so the middle of the ramp stays saturated and fades at the ends.
- Brightest swatch is **first (top)**. Chroma mode is the mirror: most
  desaturated first.

An earlier version computed the window against the in-gamut lightness span
instead, which squashed the palette at high Range. Don't "fix" it back.

## Conventions

- **One improvement per commit**, with a message explaining the why.
- Tabs for indentation in new files.
- Minimal comments — a comment survives only where the code is genuinely weird
  for a good reason.

## Branches

- `main` — the clean three-slider baseline.
- `improvements-no-square-picker` — current work: the analytical gamut solver,
  color-space registry and selector, rail improvements, multi-format copy, WCAG
  contrast, ΔE legend, plus the dormant square picker.
- `origin/claude/square-picker` — the original branch where the square picker
  *replaced* the sliders. Kept as the source of that code; the square was not
  convincing as the default, hence the dormant-module approach.

## Open threads

- Settle the square-mode UI (the toggle button is placed arbitrarily).
- Swatch border is `rgba(0,0,0,0.8)`; may be too heavy.
- A pure-black swatch reports a small nonzero chroma in the oklch readout.
- `improvements-no-square-picker` has not been merged to `main`.
