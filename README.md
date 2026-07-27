# palettable

An OKLCH color palette picker in a single self-contained `index.html` —
no build step, no dependencies. Open the file in a browser.

Pick a key color by hue, lightness and chroma, then generate a palette that
varies one of those axes. Everything stays in gamut: the chroma control is a
percentage of the exact maximum chroma available at the current lightness and
hue, solved analytically. Target gamut is selectable (sRGB, Display P3,
Rec. 2020), and swatches copy as hex, RGB, HSL, `oklch()` or `color()`.

See `CLAUDE.md` for architecture and conventions.
