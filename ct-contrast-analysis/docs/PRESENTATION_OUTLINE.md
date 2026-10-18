# Presentation Outline (12 slides, ~10 minutes)

Build the slides yourself. Each row says what goes on the slide and which output to use. Numbers are from the executed notebook.

| # | Slide | Content | Visual |
|---|---|---|---|
| 1 | Title | Title, name, course, faculty | — |
| 2 | Problem | What IV contrast does; the contrast-agent header tag is present in only 14 of 100 files here → need to tell from the image | fig 01 (contrast vs no-contrast slice) |
| 3 | Objectives | HU features · 3-way comparison · leakage-safe evaluation · beat a generic baseline · explain | — |
| 4 | Dataset | 100 → 94 slices, 54 patients, 47/47; label = curator folder; 9 patients in both classes; **approval status** | fig 02 |
| 5 | Leakage audit | File names, contrast tags, "CT CHEST WITHOUT CONTR"-type text → removed | Part 2 table |
| 6 | Preprocessing | DICOM → HU, body mask, 6 duplicates removed, outliers kept (why) | fig 01 |
| 7 | Features (novelty 1) | HU bands, CEI; contrast shifts tissue into 130–400 HU | fig 03 + fig 04 |
| 8 | Evaluation design (novelty 2) | Patient-grouped 5×10 CV; nested tuning; permutation test (p = 0.005) | diagram + fig 08 |
| 9 | Results | Metadata 0.55 · image 0.94 · combined 0.94 | figs 06, 07 |
| 10 | Baseline vs proposed | Raw pixels 0.76 → HU features 0.94 (+0.175, better in 94% of folds); tuning/ensemble: no gain | fig 11 |
| 11 | Explainability & insights | Enhancement group drives the model; scanner shortcut 0.73; errors = abdominal slices + bone | figs 09, 10, 12 |
| 12 | Limitations · conclusion · future | 3 limitations; one-sentence answer; aorta segmentation / 3-D / external test | — |

**Tips:** lead each results slide with the one number that matters; say "mean ± SD over 50 patient-level folds"; be ready to explain why metadata didn't help (VIVA_GUIDE §3).
