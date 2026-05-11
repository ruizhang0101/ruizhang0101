---
name: CS-systems paper figure conventions
description: Format, sizing, fonts, colors, and line styles for figures in CS-systems conference submissions
type: feedback
scope: area:cs-systems-papers
---

When generating or modifying figures for a CS-systems conference submission
(USENIX OSDI/SOSP/ATC/NSDI, ACM sigconf), follow these conventions.

**Format**
- Save vector PDF (`plt.savefig("fig.pdf")`) — survives reviewer zoom and prints cleanly.
- PNG only when content is genuinely raster (screenshots, dense heatmaps); use 300–600 DPI when forced to PNG.

**Sizing — match column width**
- Single-column: `figsize=(3.3, ~2.0–2.5)`
- Double-column (`figure*`): `figsize=(7.0, ~3.0)`
- Set figsize to the *final* printed size; never draw huge then shrink in LaTeX (fonts and line weights scale wrong).

**Fonts**
- Body fonts in figures should land in 8–10 pt at printed scale. Use rcParams: `font.size=9, axes.labelsize=9, xtick/ytick.labelsize=8, legend.fontsize=8`.
- Match paper body font when possible (Times for USENIX, Linux Libertine for ACM acmart).

**Color**
- Assume black-and-white printability — desaturation-check every figure.
- Colorblind-safe palettes only (Wong's 8-color, ColorBrewer Set2/Dark2, matplotlib tab10). Avoid jet/rainbow.
- ≤ 4 colors per figure. For > 4 series, reuse 2–3 hues and disambiguate with linestyle (solid/dashed/dotted) + marker (o/s/^) for redundant encoding.
- **Lock per-system colors across the paper.** Pick once, reuse everywhere. Reviewers shouldn't have to re-learn the legend per figure.

**Tufte / data-ink ratio**
- Drop top/right spines, redundant gridlines, boxed legends when avoidable.
- Direct-label lines when there are ≤ 5 series (saves a legend block).
- Data lines `linewidth=2`, axis spines `linewidth=0.5` so data is visually heavier than chrome.
- No 3D, gradients, drop shadows, ornamental backgrounds.

**Bars**
- Error bars (stdev or 95% CI) on every bar with ≥ 3 seeds — reviewers ask.
- Annotate values on bars when ≤ 6 bars per group.
- Order by value within group when rank is the message; keep fixed order when across-group comparison is the message.

**Axis ticks**
- Force integer ticks on integer-valued axes (`MaxNLocator(integer=True)`) to avoid "0.5 messages" tick artifacts.

**Why:** USENIX prints submissions in B&W and reviewers explicitly check this; ACM re-renders fonts for camera-ready so vector PDFs survive better than PNGs; Tufte/data-ink reduces clutter that reviewers flag as chartjunk; locked per-system colors stop reviewers having to re-learn the legend each figure.

**How to apply:** Verify the conventions before declaring a figure done; flag intentional deviations explicitly. Run a desaturation check (open the PDF in a B&W preview, or convert with `gs -sDEVICE=pdfwrite -sColorConversionStrategy=Gray`).
