# Worm Track Analyzer

A single-file web app for analysing *C. elegans* locomotion and chemotaxis tracks. Open the HTML
file in a browser, drop in your tracking output, and get reorientation rates, speed, run statistics,
gradient navigation, roaming/dwelling, MSD and salt-response plots — with the statistics, the figures
and the raw numbers ready to export.

**[Open the app →](https://anjaliav.github.io/Chemotaxis-Analysis-Webapp/)**

Nothing is uploaded anywhere. The whole program is one `index.html` with no libraries and no build
step; it runs entirely in the browser, works offline, and your data never leaves your computer.

---

## Quick start

1. Open the link above, or download `index.html` and double-click it.
2. In **Data & groups**, create each condition and add its CSV or Excel recordings.
3. Set the fallback frame rate, select the spatial calibration mode, and enter each recording’s goal
   and plate centre on the same tab. Regression bearing assignment requires a frame rate.
   Use **Distance axes** to display tracks and distances in pixels.
4. Open **Tracks** and adjust the turn thresholds until the marks line up with the turns
   you can see.

There is a **Load demo data** button if you want to see the whole thing working before you commit
your own files, and a **Guide for beginners** tab written for someone meeting worm tracking for the
first time.

When recorded scales span at most 5 pixels/mm, the app averages their pixel sizes in µm/pixel
and converts the mean into one scale for all conditions. A manually entered scale overrides
automatic averaging.

Under **Data & groups → Calibration → Spatial calibration**, select **Use each recording’s
reported scale** to convert each video’s tracks and landmarks to millimetres using its scale override
or reported scale. The fallback applies when both are blank. If some recordings still need a scale,
the app lists them and excludes them from millimetre-based analysis.

In **Data & groups**, each recording’s **FPS override** takes precedence over its file metadata
and the shared fallback frame rate. Recordings with known rates are compared in seconds. If some
rates are missing, those recordings are excluded from the analysis until a rate is provided.
When all rates are missing, analysis uses frames.

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
- Heading distributions on a rose plot, in plate coordinates or relative to the cue, with the
  resultant vector R, the circular mean and a Rayleigh test
- A modelled salt gradient, and reorientation rate against concentration, ΔC, dC/dt and % change
- Tracks and reorientation locations drawn over the modelled gradient field

**Statistics.** Plate / file is the default replicate unit. With two conditions, the plate
comparisons for runs, total distance, overall reorientation, and bearing slopes report a Welch
test and a 95% confidence interval for the mean difference. They assume independent plates.
Other comparisons retain the Welch and Mann–Whitney tests for two conditions, and ANOVA or
Kruskal–Wallis with Holm-adjusted post hoc comparisons for three or more.

**Bearing.** The selected curve uses Method B turns, a 3 s heading regression ending 1 s before
estimated onset, eligible-run exposure, and five bearing bins by default. Each bin rate divides
pooled worm event counts by pooled exposure time on that plate. The curve shows the plate mean and
pointwise 95% confidence intervals. Files with the same plate ID and experimental day pool for
bearing. Each plate with at least three bins with exposure contributes one slope. The slope plot can
overlay worm fits, and the analysis supports day adjustment and exploratory bin comparisons with
Holm-adjusted p-values. Rejected bearings retain their original detected-event counts. Navigation
also shows the legacy curve, event inspection, and sensitivity and frame-rate checks. See the
**Methods** tab for the estimator and exposure definitions.

**No-gradient controls.** Mark a condition as *No gradient* and it is not skipped — it goes through
identical code against the position the cue would have occupied. Every navigation and salt measure is
additionally tested against zero, so the claim you make is "the gradient condition differs from zero
and the control does not", which is stronger than the two merely differing.

---

## Recording landmarks and maps

Enter each recording’s goal and plate centre in **Data & groups**, in original video pixels.
Excel files with a `start-end coords` sheet fill the recording start and salt peak automatically.
The **Mark on plot** control edits one recording at a time. Tracks align to the recording start
when available and otherwise to each worm’s first position. Whole-plate maps align recording centres
using the diameter under **Salt gradient**. Recordings with missing centres are listed and can
be viewed in **Track region**.
Rotation uses the centre-to-goal axis for plate-aligned maps. **One panel per recording**
centres each field on that recording’s goal, imported as the **End Point** in Excel. Pooled
maps centre the field on the mean goal and show individual goals as open circles. The solid
rim is the plate boundary. The dashed ring marks the worm-placement radius in millimetres.
Salt and bearing calculations use the original recording coordinates. Maps use millimetres
when the plotted recordings have different spatial scales.

On **Gradient map**, **Reorientation colour → By bearing (Viridis)** colours reorientation
starts by the bearing just before the turn, from −1 (away from the goal) to +1 (toward it).
The figure exports include the bearing legend, and the reorientation CSV includes the value
for each turn.

---

## Exports

Everything the app shows can leave it.

- **Per figure** — `CSV` provides wide-format data with separate columns for each condition.
  `Plate & worm CSV` on the runs, total distance, overall reorientation, and bearing slope plots
  lists each plate followed by its worms. Filter the `Unit` column to select either level.
  The slope export includes fit coverage for both levels and leaves unavailable slopes blank.
  `Plate bins CSV` on the bearing curve includes counts and exposure time.
  Figures are also available as `SVG`, `PNG`, and `PDF`.
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
index.html                             the current application
Worm_Track_Analyzer_Chemotaxis.html    earlier standalone analyzer snapshot
README.md                              this file
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
  bolus, with no plate wall: it underestimates the field near the rim, and the more so the older the
  gradient. The first minutes after a drop are underestimated too, unless a drop radius is given.

## Licence

Not yet chosen — add one before sharing widely (MIT is the usual choice for a tool like this).
