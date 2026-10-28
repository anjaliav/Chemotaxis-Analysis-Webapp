# Worm Track Analyzer

A single-file web app for analysing *C. elegans* locomotion and chemotaxis tracks. Open the HTML
file in a browser, drop in your tracking output, and get reorientation rates, speed, run statistics,
gradient navigation, roaming/dwelling, MSD and salt-response plots — with the statistics, the figures
and the raw numbers ready to export.

**[Open the app →](https://anjaliav.github.io/Chemotaxis_Tracks_Analysis_Webapp_v1/](https://anjaliav.github.io/Chemotaxis-Analysis-Webapp/)**

Nothing is uploaded anywhere. The whole program is one `index.html` with no libraries and no build
step; it runs entirely in the browser, works offline, and your data never leaves your computer.

---

## Quick start

1. Open the link above, or download `index.html` and double-click it.
2. On the **Tracks** tab, drag your CSV or `.xlsx` files onto the drop zone — one file per plate.
3. On **Data & groups**, add a second condition and drop its files in.
4. On **Analysis settings**, set the **frame rate** (fps) and the **scale** (pixels per mm). Without
   them the app still works, but time is counted in frames and distance in pixels.
5. Go back to **Tracks** and move the threshold sliders until the marks land on the turns you can
   actually see. That calibration step is the analysis; everything downstream follows from it.

There is a **Load demo data** button if you want to see the whole thing working before you commit
your own files, and a **Guide for beginners** tab written for someone meeting worm tracking for the
first time.

---

## Input formats

| Input | What it needs |
|---|---|
| **CSV / TSV** | Columns `WormID`, `Frame`, `PositionX`, `PositionY`; `Quality` optional. Common aliases (`worm`, `track_id`, `x`, `y`, `centroid_x`, …) are recognised. |
| **Excel `.xlsx`** | A `Tracks` sheet with the same columns. A `Parameters` sheet fills in frame rate and scale automatically; a `Centerlines` sheet (ten body points per frame) unlocks posture-based turn detection. |
| **Whole folder** | Drop a folder and it picks up the track files plus `worm_summary.json` for calibration, and keeps companion files in the exported archive. |
| **Project `.zip`** | An archive exported by the app — restores every file, condition, colour and threshold. |

One file is treated as one replicate (plate). Quality codes are exporter-specific, so the app shows
you the percentage of rows carrying each code and lets you choose which to keep rather than assuming.

---

## What it computes

**Reorientations — three detectors, so you can check one against another**

- **Method A** — pirouette detection on turning rate: |ω| above a threshold, events merged within
  *t*<sub>crit</sub> (after Pierce-Shimomura et al., 1999).
- **Method B** — heading change across a fixed window with refractory non-maximum suppression
  (*the default*).
- **Method C** — posture-based, when centerlines are available: body-axis sign flips give reversals,
  tip-to-tip collapse relative to body length gives omega turns.

**Everything else**

- Speed (mean, median, or run speed with turns excluded), total distance, net displacement, path efficiency
- Run duration and run distance between consecutive reorientations, with distributions
- Navigation in a gradient: turn rate versus bearing (cos θ), weathervaning, change in bearing across
  a turn, and a rigid-rotation shuffle null
- Roaming and dwelling from log₁₀(speed ÷ angular speed), threshold by Otsu or by hand
- Mean squared displacement with the log–log exponent α and an effective diffusion coefficient
- Time course of any of the above in equal time bins
- A modelled salt gradient, and reorientation rate against concentration, ΔC, dC/dt and % change
- Tracks and reorientation locations drawn over the modelled gradient field

**Statistics.** Two groups: Welch's *t* and Mann–Whitney *U*. Three or more: one-way ANOVA with
pairwise Welch tests, and Kruskal–Wallis with Dunn's post hoc — all Holm-corrected. A D'Agostino K²
screen decides which family is reported as primary. The replicate unit is switchable between
individual worm and plate; plate is the conservative choice when you have several plates per
condition. The statistics library was verified digit-for-digit against SciPy.

**No-gradient controls.** Mark a condition as *No gradient* and it is not skipped — it goes through
identical code against the position the cue would have occupied. Every navigation and salt measure is
additionally tested against zero, so the claim you make is "the gradient condition differs from zero
and the control does not", which is stronger than the two merely differing.

---

## The salt gradient model

The field is modelled as a point bolus diffusing in a thin agar layer:

```
C(r) = C_background + Σ_drops  1e6 · N / √(4·π·h·D·t) · exp( −r² / (4·D·t) )

N = (stock_mM / 1000) · (volume_µL / 1e6)              moles of NaCl in the drop
h = agar_volume_mL / (π · plate_radius_mm²) · 100      agar height, cm
D = 1.516e-5 cm²·s⁻¹                                   NaCl, Robinson & Stokes (1951)
t = seconds since that drop was placed
```

Nothing in it is fitted to behaviour. Single-Gaussian, exponential and linear-ramp alternatives are
selectable so a result can be shown not to depend on the shape of the model. The **Methods** tab
prints the formula, the live JavaScript, and a standalone Python file with your own parameters baked
in — download it and check the numbers independently.

Two things worth knowing before quoting salt results: ΔC and dC/dt are *differences*, so the assumed
background concentration cancels exactly, while the percentage-change axis divides by concentration
and inherits it. And concentration is one-to-one with distance from the drop, so a rate-versus-C plot
is also a map of where the animal was — which is what the no-gradient control is for.

---

## Exports

Everything the app shows can leave it.

- **Per figure** — a button row under every panel: `CSV` (the numbers behind that figure, wide
  format), `SVG`, `PNG`, `PDF`.
- **`csv/`** — every table in long format, one file each, for scripting.
- **`wide/column/`** — one column per condition, one row per replicate. Paste into GraphPad Prism as
  a Column table.
- **`wide/xy/`** — X, then mean / SEM / N per condition. Paste into Prism as an XY table.
- **`wide/by_figure/`** — one file per figure, named after the figure, for assembling supplementary data.
- **`figures_svg/`, `figures_pdf/`, `figures_png/`** — vector and high-resolution raster copies,
  including `ALL_figures.pdf` with every panel on its own page behind a cover page listing the settings.
- **`source_data/<condition>/`** — the files you loaded, with a manifest, plus `project.json` and
  `analysis_settings.json`.

Drop that archive back onto the app and the whole session returns — same files, same conditions, same
thresholds, identical numbers. That round-trip is tested: every computed value matches to six decimal
places after a reopen.

---

## Reproducibility notes

- Figures are true vectors (the PDF writer is hand-rolled, Base-14 fonts, no rasterisation), so they
  stay editable in Illustrator or Inkscape.
- Every exported archive carries a `README.txt` listing every threshold used to produce it.
- Thresholds are a judgement call, and the app makes that judgement visible: the diagnostics show the
  distribution of turning rates and heading changes with the threshold drawn on them, a sensitivity
  sweep of detected rate against threshold, and the directional-persistence curve that tells you what
  sampling interval your data can support.

---

## Browser support

Any current Chrome, Edge, Firefox (113+) or Safari (16.4+). Reading `.xlsx` needs
`DecompressionStream`, which is why those minimum versions apply; CSV works everywhere.

## Repository layout

```
index.html    the entire application
README.md     this file
```

## Citing

If this tool contributed to published work, please cite the repository and state the detection method
and thresholds you used (Methods tab → settings, or the `README.txt` in your exported archive).
Methods implemented here follow, among others:

- Pierce-Shimomura JT, Morse TM, Lockery SR (1999) *J Neurosci* **19**:9557–9569
- Iino Y, Yoshida K (2009) *J Neurosci* **29**:5370–5380
- Ben Arous J, Laffont S, Chatenay D (2009) *PLoS ONE* **4**:e7584
- Robinson RA, Stokes RH (1951) *J Am Chem Soc* **73**:4332
- Kramer TS et al. (2026) *Nat Neurosci* — bearing defined as cos θ

## Known limitations

- Centroid-based detectors cannot separate a reversal from an omega turn; Method C can, but needs centerlines.
- A worm that reverses along its own path produces almost no centroid displacement and can be missed.
- Collisions and track swaps produce spurious hairpins; the jump filter catches the worst, but tracks
  with many crossings deserve a look.
- The salt model assumes a laterally infinite, uniformly thick agar layer and an instantaneous point
  bolus — it is wrong within a few millimetres of the rim and in the first minutes after a drop.

## Licence

Not yet chosen.
