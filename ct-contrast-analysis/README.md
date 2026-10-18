# CT Image Analysis Using Machine Learning with Patient Metadata

B.Tech Healthcare Analytics mini-project. One Jupyter/Colab notebook that inspects a public CT dataset, converts the images to Hounsfield Units (HU), extracts physics-based image features, and answers:

> **Can we tell from a single CT slice whether IV contrast was given — and does patient metadata (age, sex) help beyond the image?**

> ⚠️ **Dataset approval:** the course requires a dataset from the Moodle list or prior approval. This Kaggle dataset is **not yet confirmed as approved** — see `docs/DATASET.md`.

## Dataset (verified from the actual files)

[CT Medical Images — kmader/siim-medical-images](https://www.kaggle.com/datasets/kmader/siim-medical-images): middle CT slices from the TCGA-LUAD collection of The Cancer Imaging Archive. Not included in this repository.

| | |
|---|---|
| Records | 100 slices in `overview.csv` → **94** after removing 6 pixel-identical duplicates |
| Patients | **54** (`PatientID`); 1–6 slices each; **9 patients have slices in both classes** |
| Target | `Contrast` (True/False) — 47 / 47 after de-duplication |
| Label origin | Equals the curator's folder name (`Contrast` / `No contrast`) for every row |
| Patient metadata | Age and sex (100% complete). Weight, height, smoking, ethnicity: 6–42% present → not used |
| Not the label | `ContrastTag` (sparse DICOM agent field: `NONE` in 36 contrast slices) |

Full details: `docs/DATASET.md`.

## Repository layout

```
ct-contrast-analysis/
├── notebooks/CT_Contrast_Analysis.ipynb   # whole pipeline, executed on the real data (outputs included)
├── results/                               # tables + figures produced by that run
│   ├── baseline_vs_proposed.csv, baseline_cv_summary.csv, nested_tuning_comparison.csv, ...
│   ├── run_summary.json, environment.json
│   └── figures/01_…12_*.png
├── docs/
│   ├── DATASET.md               # verified dataset facts, licence, approval flag
│   ├── REPORT_GUIDE.md          # 19-section structure + your findings to interpret
│   ├── PRESENTATION_OUTLINE.md  # 12 slides
│   ├── VIVA_GUIDE.md            # every step explained + likely questions
│   └── REFERENCES.md            # verified citations
├── data/                                  # put the dataset here (git-ignored)
├── requirements.txt
└── .gitignore
```

## How to run

**Google Colab.** Upload the notebook. Then either drag Kaggle's `archive.zip` into the *Files* panel (the notebook unzips it), or set `USE_KAGGLEHUB = True`. *Runtime → Run all.* CPU is enough.

**Local Jupyter.**
```bash
pip install -r requirements.txt
# put archive.zip next to the notebook, or unzip it into data/siim-medical-images/
jupyter notebook notebooks/CT_Contrast_Analysis.ipynb
```

**Troubleshooting.** `overview.csv not found` → the zip isn't where the notebook looks (see Part 0). Pixel-data errors → `pip install pylibjpeg pylibjpeg-libjpeg`.

## Pipeline

| Part | What happens |
|---|---|
| 1 Inspection | Folder inventory, CSV columns, label provenance, DICOM tag census, patient IDs, sparse-field check, TIFF/NPZ check |
| 2 Leakage audit | Excludes file names, contrast tags and free-text descriptions that state the answer (e.g. "CT CHEST WITHOUT CONTR") |
| 3 Preprocessing | DICOM → HU, padding/clipping, body mask, unreadable-file handling, duplicate removal |
| 4 Image features | 25 HU features inside the body mask: tissue fractions, **Contrast Enhancement Index (CEI)**, enhanced-region size, HU percentiles/moments, GLCM texture, edges |
| 5 Metadata & quality | Age, sex; missing values, outliers (kept, with reason), encoding, scaling, class balance |
| 6 EDA | Class/age/patient plots, per-class HU histograms, effect sizes, correlation heatmap |
| 7 Models | LogReg / Random Forest / SVM × {metadata, image, combined}; `StratifiedGroupKFold` by patient, 5-fold × 10 repeats; scanner-settings shortcut test |
| 8 Improvement | Nested selection + tuning; patient-grouped permutation test; single- and group-level permutation importance; HU overlays + label audit; post-hoc bone-suppressed features; soft-voting ensemble; baseline-vs-proposed table |

## Results (executed on the real dataset, 10 Oct 2026)

Mean ± SD over **patient-level** cross-validation folds (5-fold × 10 repeats unless noted). Source: `results/baseline_vs_proposed.csv`.

| Approach | Type | ROC-AUC | Bal. acc. | Sensitivity | Specificity | F1 |
|---|---|---|---|---|---|---|
| Metadata only — age, sex (RF) | Baseline | 0.554 ± 0.137 | 0.533 | 0.623 | 0.444 | 0.556 |
| Raw pixels → PCA → LogReg | Baseline | 0.762 ± 0.138 | 0.702 | 0.728 | 0.677 | 0.705 |
| **HU features (LogReg)** | **Proposed** | **0.938 ± 0.053** | **0.894** | **0.848** | **0.939** | **0.885** |
| HU features + metadata (RF) | Proposed | 0.937 ± 0.056 | 0.874 | 0.833 | 0.915 | 0.865 |
| + nested selection & tuning (LogReg, 3 repeats) | Proposed | 0.922 ± 0.059 | 0.843 | 0.792 | 0.894 | 0.834 |
| + soft-voting ensemble | Proposed | 0.938 ± 0.056 | 0.883 | 0.816 | 0.949 | 0.868 |
| Bone-suppressed enhancement, 4 features (LogReg) | Post-hoc | 0.950 ± 0.046 | 0.884 | 0.820 | 0.947 | 0.870 |

Other verified results:
- Proposed HU features vs raw-pixel baseline on identical folds: **+0.175 AUC, better in 94% of folds**.
- Permutation test: real AUC 0.903 vs 200 label-shuffled runs, **p = 0.005** — the smallest value possible with 200 shuffles (no shuffled run reached the real score).
- Feature-group importance (held-out AUC drop when shuffled): enhancement group **0.214**, HU percentiles 0.050, metadata ≈ 0, texture ≈ 0.
- Each group alone: enhancement 0.932, percentiles 0.890, texture 0.837, other fractions 0.622, metadata 0.552.
- Scanner/protocol settings alone (shortcut test): **AUC 0.73 ± 0.12**.

## Key findings (factual summary — interpret them in your own words)

1. **The image carries the signal; age and sex do not.** Metadata alone is near chance, and adding it to the image features changes AUC by ≈ 0.
2. **Physics-based HU features beat a generic pixel pipeline by a wide margin** (0.94 vs 0.76) on the same folds.
3. **The Contrast Enhancement Index is the single most useful feature**, and the enhancement group explains most of the model's performance — what CT physics predicts, since iodine raises blood density into ~130–400 HU.
4. **Tuning and the ensemble did not help.** Plain logistic regression was already at what ~94 slices support.
5. **Scanner settings partly predict contrast (AUC 0.73)** — a shortcut risk; they are kept out of the main models.
6. **The confident errors trace to anatomy rather than obvious label noise.** Upper-abdomen slices show an enhanced aorta while a large liver dilutes the enhanced fraction, and bone shares the HU band. The bone-suppressed refinement gave a 4-feature model with similar AUC, but the gain is within noise (better in 50% of folds, worse in 40%) and was designed after seeing the errors.

## What is novel compared with the baselines

- **Generic baseline** = 8-bit pixels → PCA → classifier: ignores physical units, anatomy and patients. **This project** computes features in Hounsfield Units inside a body mask, built around how iodinated contrast changes blood density (the CEI).
- **Leakage-aware evaluation**: an explicit audit of file names, DICOM tags and free text; duplicate removal; patient-grouped repeated CV — essential here because 9 patients appear in both classes.
- **Honest improvement testing**: nested tuning, a patient-grouped permutation test, group-level explainability, a scanner-shortcut check, and an error-analysis loop with its post-hoc caveat stated.

The contribution is careful, physically grounded, leakage-safe analysis — not a new algorithm, and not a claim of clinical validity.

## Limitations

- 94 slices / 54 patients, one 2-D slice per series → wide uncertainty; no external test set.
- Labels follow the curator's folders, not a radiologist's per-image reading; contrast **phase** (arterial / venous / delayed) is unknown.
- Slices come from different body levels (neck to pelvis), which changes the HU mix independently of contrast.
- Bone overlaps the enhanced-blood HU band; the post-hoc suppression misses vertebral marrow when the rim is open in 2-D.
- Patient metadata is limited to age and sex; richer fields are mostly empty.
- Scanners/protocols differ between classes (shortcut AUC 0.73), so some image differences may reflect protocol rather than contrast alone.

## Future scope

- Segment the aorta and measure its HU directly (independent of slice level).
- Use full 3-D series from TCIA instead of one slice; add contrast-phase labels.
- External validation on another TCIA collection; compare with pretrained-CNN embeddings once more data is available.

## Verification status

| What | Status |
|---|---|
| Notebook on the real dataset | ✅ Executed end-to-end without errors (≈ 6 min on 2 CPU cores). Environment: Python 3.13, pandas 3.0, scikit-learn 1.9 — `results/environment.json` |
| Google Colab | ⚠️ **Not executed in Colab by me.** Colab ships older library versions; the code avoids version-specific APIs, but run it once in Colab and compare with `results/` |
| Reproducibility | Fixed seeds; two separate full runs produced identical result tables |

Educational project — not a medical device.
