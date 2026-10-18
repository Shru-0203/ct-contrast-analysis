# Report Guide — 19 sections

The faculty requires the report to be your own, AI-free writing. This guide gives the **structure**, **what each section must contain**, and **where each fact or number comes from**. It contains no ready-made sentences. Write the interpretation yourself.

Figures: `results/figures/NN_*.png`. Tables: `results/*.csv`. Summary: `results/run_summary.json`.

| # | Section | Must contain | Source |
|---|---|---|---|
| 1 | **Title Page** | Title, name, roll no., course, faculty (Dr. Prerna), institute, date | — |
| 2 | **Abstract** (150–250 words) | Problem → data (94 slices, 54 patients) → method (HU features, patient-level CV) → headline numbers → one-line conclusion | `baseline_vs_proposed.csv` |
| 3 | **Introduction** | CT and IV contrast; why contrast status matters; why header fields can't be trusted (agent tag present in only 14 of 100 files here) | VIVA_GUIDE §1; REFERENCES [4] |
| 4 | **Problem Statement** | Input (one CT slice + age/sex), output (`Contrast`), constraints (small data, no leakage) | Part 1.4 |
| 5 | **Objectives** | 4–6 measurable ones (HU features; 3-way comparison; leakage-safe evaluation; beat a generic baseline; explain predictions) | — |
| 6 | **Literature Review** | 4–8 works you've *read*: [4] contrast detection, [5] texture, [7] leakage, [8] nested CV, [9] permutation tests, [10] shortcuts. For each: what they did, what you use, how yours differs | `REFERENCES.md` |
| 7 | **Dataset Description** | Files, counts, columns, label origin, `ContrastTag` caveat, patients (9 in both classes), duplicates, sparse fields, licence, **approval status** | `DATASET.md`; Part 1 outputs |
| 8 | **Data Preprocessing** | HU conversion, padding, body mask, duplicates (6 removed), missing values, outliers (kept + why), encoding, scaling, imbalance, splitting | Parts 3, 3b, 5b; fig 01 |
| 9 | **Proposed Methodology** | Block diagram; feature table; models; CV design; metrics | Parts 4–7 |
| 10 | **Novelty / Innovation** | Only the claims listed below | README "What is novel" |
| 11 | **Implementation** | Environment + versions table, how to run, runtime, repo structure, Colab caveat | `environment.json`, README |
| 12 | **Experimental Results** | 3-way comparison table (mean ± SD), nested tuning, ensemble, permutation test | `baseline_cv_summary.csv`, `nested_tuning_comparison.csv`, fig 08 |
| 13 | **Performance Evaluation** | All metrics incl. specificity; ROC; confusion matrices; fold spread | figs 06, 07 |
| 14 | **Visualizations & Healthcare Insights** | Each figure: what it shows + clinical meaning | figs 02–05, 09, 09b, 10, 12 |
| 15 | **Baseline Comparison** | Metadata-only and raw-pixel baselines vs each proposed step; paired-fold statement | `baseline_vs_proposed.csv`, fig 11 |
| 16 | **Discussion** | Why each result happened; shortcut check; label audit; post-hoc caveat; limitations | Parts 7.4, 8.4, 8.4b |
| 17 | **Conclusion** | Answer the problem statement with your numbers; nothing new | — |
| 18 | **Future Scope** | 2–4 concrete, justified steps | README |
| 19 | **References** | Only sources you opened; one consistent style | `REFERENCES.md` |

## Your findings to interpret (verified numbers from the executed notebook)

Write *why* each happened and *what it means clinically* in your own words.

| Finding | Number | Where |
|---|---|---|
| Metadata alone | AUC 0.554 ± 0.137 | Part 7 table |
| Raw-pixel baseline | AUC 0.762 ± 0.138 | Part 8.5 |
| HU features (best, LogReg) | AUC 0.938 ± 0.053; sensitivity 0.848; specificity 0.939; F1 0.885 | Part 7 / 8.5 |
| HU + metadata | AUC 0.937 ± 0.056; ΔAUC vs image-only ≈ 0 | Part 7 paired lines |
| Proposed vs raw-pixel | +0.175 AUC, better in 94% of folds | Part 8.5 |
| Nested tuning | 0.922 (no gain) | Part 8.1 |
| Ensemble | 0.938 (no gain) | Part 8.5 |
| Post-hoc bone suppression | 0.950 ± 0.046; better in 50%, worse in 40% of folds | Part 8.4b |
| Permutation test | real 0.903, p = 0.005 (200 shuffles) | fig 08 |
| Group importance | enhancement 0.214; percentiles 0.050; metadata ≈ 0 | fig 09 |
| Group alone | enhancement 0.932; metadata 0.552 | fig 09b |
| Scanner shortcut | AUC 0.73 ± 0.12 | Part 7.4 |
| Age by class | median 71 vs 69, p = 0.104 | Part 6 |
| Most confident errors | upper-abdomen contrast slices; bone in HU band | figs 10, 12 |

## Novelty claims — what you can and cannot say

**Supported by your run:**
- Physics-based HU features with an engineered Contrast Enhancement Index, which beat a generic raw-pixel pipeline on identical patient-level folds.
- A leakage audit that found and removed answer-revealing fields (file names, contrast tags, free-text descriptions) and duplicates.
- Patient-level repeated CV (needed: 9 patients in both classes), nested tuning, and a permutation test against chance.
- Explainability at the feature-group level, showing the enhancement features drive the model, consistent with CT physics.
- An error-analysis loop (label audit → bone-suppression refinement), reported with its post-hoc caveat.

**Not supported — do not claim:**
- That metadata, tuning or the ensemble improved results (they didn't).
- That bone suppression is a proven improvement (within noise, post-hoc).
- Clinical validity or readiness; superiority over published deep-learning systems.

## Writing checks
- Always "mean ± SD over N patient-level folds".
- Each figure has a caption saying what to notice.
- Keep the limitations section honest: small data, folder-based labels, unknown phase, slice level, bone overlap, scanner shortcut, no external test set.
- Dataset approval: state its status explicitly.
