# Viva Guide — understand and defend every step

Plain-language notes on what each part of the notebook does, why, and what it found on the real data. Numbers come from the executed notebook and `results/`. If you re-run, quote **your** outputs.

---

## 1. The problem in one paragraph

Radiologists inject an iodine-based **contrast agent** so blood vessels and organs stand out on CT. Knowing whether a scan is contrast-enhanced matters for reading it correctly, for building research datasets, and for quality control. The DICOM header field meant to record it is often empty: in this dataset it is present in only 14 of 100 files, and 2 of those are blank. The project asks whether the **image itself**, plus basic **patient metadata** (age, sex), can tell us, and how much each source contributes.

**Why this task?** The dataset has no tumour labels or annotations. Its only clinically meaningful label is the contrast flag (verified in Part 1).

---

## 2. Step by step

### Part 1 — Inspection (verified facts)
- 100 slices, all 512×512; 54 patients; 50/50 contrast labels.
- **Label provenance:** `Contrast` equals the folder name in `raw_input_path` (`Contrast` / `No contrast`) for all 100 rows. The label reflects how a curator sorted the series, not a radiologist reading each image.
- **`ContrastTag` is not the label.** It copies the sparse DICOM agent field and says `NONE` for 36 contrast slices.
- **9 patients have slices in both classes** (e.g. a plain scan and a contrast scan).
- Age in the CSV equals the DICOM age for all rows. 6 patients have different ages across their slices, meaning scans from different years.
- Only age and sex are complete. Weight (24%), height (6%), smoking (9%) and ethnicity (6%) are too sparse to use.
- TIFFs are HU copies of the DICOMs (≤ 1 HU difference). The `.npz` holds pickled Python objects, so it is **not loaded**: unpickling a downloaded file can run arbitrary code.
- **Defend it:** "I didn't assume anything. I checked where the label came from, which fields are complete, and which files are duplicates before choosing the task and the features."

### Part 2 — Leakage audit
- **Leakage** = features that give away the answer, or that wouldn't exist at prediction time.
- Found: file names (`..._CONTRAST_1_...`), `raw_input_path` (folder name), DICOM contrast tags, and free-text fields such as `StudyDescription` "CT CHEST WITHOUT CONTR", `RequestedProcedureDescription` "71260 CT Chest with contrast", and `CodeMeaning`. All excluded.
- **Defend it:** "A model reading 'CT CHEST WITH CONTRAST' would be cheating. Only age and sex, plus features computed from pixels, go into the models."

### Part 3 — Hounsfield Units, body mask, duplicates
- `HU = raw × RescaleSlope + RescaleIntercept`, a physical scale: air −1000, fat ≈ −100, water 0, soft tissue 20–80, contrast-filled blood ≈ 150–400, bone 400+.
- Scanner padding (raw −2000) is set to air; values are clipped to [−1024, 3071].
- **Body mask:** threshold −500 HU → opening → largest connected region (the patient, not the table) → fill holes (lungs count as inside). Figure 01 shows it working on real slices.
- **Duplicates:** 6 pairs of pixel-identical slices (same patient, same label), found by MD5 hash → 94 unique slices.
- **Error handling:** unreadable files are skipped and listed.

### Part 4 — Image features (25)
| Feature | Meaning | On the real data |
|---|---|---|
| `CEI` (Contrast Enhancement Index) | enhanced ÷ (soft + enhanced) tissue | strongest single separator (rank-biserial r = +0.87; top permutation importance) |
| `frac_enhanced` | share of body pixels in 130–400 HU | r = +0.83 |
| `tis_p90`, `tis_kurt` | upper HU percentile; peakedness of the tissue HU distribution | r = +0.80, −0.77 |
| `enh_*` | size / number of enhanced regions | aorta and heart are large enhanced blobs |
| `glcm_*`, `edge_mean` | texture (Haralick GLCM), edge strength | weak on their own |
| `body_area_frac` | patient size / field of view | confounder check — weak |

**GLCM** counts how often grey level *i* sits next to grey level *j*. "Contrast" is high when neighbours differ; "homogeneity" is high when they're similar. Background pairs are removed.

### Part 5 — Metadata and data quality
- Metadata = age, sex (M→1, F→0). Missing values: none. Medians are imputed inside each training fold anyway, as a safeguard.
- Outliers (beyond 3×IQR) appear in a handful of features. They are **kept**: in CT they are usually real anatomy, and RF is insensitive to them.
- Classes: 47/47 after de-duplication → balanced. `class_weight='balanced'` and balanced accuracy are still used.
- Scanner settings (kVp, slice thickness, tube current, body part, manufacturer, kernel…) are tested separately as a **shortcut check**, never in the main models.

### Part 6 — EDA (what the plots show)
- **Age:** median 71 (no contrast) vs 69 (contrast), Mann-Whitney p = 0.104 → no clear difference.
- **HU histogram (fig 03):** contrast slices have clearly more tissue in 130–400 HU and less in the soft-tissue peak. This is the physics the features target.
- **Correlation heatmap (fig 05):** many features overlap, which motivates feature selection and group importance.

### Part 7 — Models and evaluation
- **StratifiedGroupKFold:** each patient's slices stay in one fold. Essential here because 9 patients appear in both classes.
- **5-fold × 10 repeats** → mean ± SD across 50 folds.
- Imputation and scaling happen inside the pipeline (fitted on training folds only).
- **Metrics:** precision, recall (sensitivity), specificity, F1, accuracy, balanced accuracy, ROC-AUC; confusion matrices from out-of-fold predictions.
- **Results:** image-only LogReg AUC 0.938 ± 0.053; combined RF 0.937; metadata-only best 0.554. Adding metadata: ΔAUC ≈ 0 (LogReg slightly worse in 48% of folds).
- **Shortcut check:** scanner settings alone reach AUC 0.73. Body-part tag "LUNG" is more common in no-contrast slices, and some scanner models appear almost only in one class.

### Part 8 — Improvement and explanation
- **Nested CV tuning + `SelectKBest`:** AUC 0.922 vs 0.938 untuned. **No gain.** Report it; with ~94 slices the simple model is already at the limit, and tuning adds variance.
- **Permutation test:** real AUC 0.903 vs a null distribution centred near 0.5; p = 0.005 (the floor with 200 shuffles). Labels are shuffled per slice because some patients have both labels; the folds stay patient-grouped.
- **Permutation importance:** CEI is top. **Group importance:** shuffling the enhancement group drops AUC by 0.214; metadata ≈ 0.
- **Group alone:** enhancement features alone give 0.932, almost the full model.
- **Label audit (fig 10):** the most confident errors are mostly contrast slices from the **upper abdomen**. The aorta is enhanced, but the large liver/spleen dilutes the enhanced fraction. Bone is painted "enhanced" in every slice.
- **Post-hoc bone suppression (8.4b, fig 12):** 4 features → AUC 0.950 ± 0.046, but better in only 50% of folds vs the original. It was designed after seeing errors, so it is **optimistic** and labelled "post-hoc". It also misses vertebral marrow when the rim is open in 2-D.
- **Ensemble (soft voting LogReg + RF + SVM):** 0.938 — **no gain** over plain LogReg.
- **Baseline vs proposed (fig 11):** raw-pixel PCA 0.762 → HU features 0.938; better in 94% of identical folds.

---

## 3. Likely viva questions

**Q: What exactly is novel?**
Not a new algorithm. Three things: (1) features in physical Hounsfield Units, built around how iodine changes blood density, including the CEI; (2) a leakage-aware evaluation (audit of names, tags and free text; duplicate removal; patient-grouped repeated CV); (3) honest improvement testing (nested tuning, permutation test, group explainability, shortcut check, error-analysis loop). On identical folds it beats a generic raw-pixel pipeline by +0.175 AUC.

**Q: Your title says "with patient metadata", but metadata didn't help. Is that a failure?**
No, it is a finding. Age and sex have no physical link to whether contrast was injected, and the data confirm it: metadata alone is near chance, and adding it changes AUC by ≈ 0. Also, only age and sex were complete enough to use.

**Q: Tuning and the ensemble didn't improve results. Why include them?**
They were required tests of whether more complexity helps. With 94 slices it doesn't; the simplest model is as good. Reporting that honestly is better than hiding it.

**Q: How did you prevent leakage?**
Five layers: removed fields that state the answer; removed duplicates; grouped folds by patient; fitted scaling, imputation and selection inside training folds only; nested CV for tuning.

**Q: Why are there 94 slices, not 100?**
Six slices were exact pixel copies of others. Keeping them would double-count those images.

**Q: Where do the labels come from, and are they reliable?**
From the folder each series was sorted into by the dataset curator. That is not a per-image radiologist reading, and contrast phase is unknown. The label audit didn't find clear mislabels. The errors are explained by anatomy (abdominal slices) and bone overlap.

**Q: What does AUC 0.73 for scanner settings mean?**
Protocol alone partly predicts contrast, so a model could "cheat" via scanner style. That's why scanner settings aren't in the main models, and why I list it as a limitation: some image differences could be protocol-related.

**Q: Why not deep learning?**
94 slices from 54 patients is far too few to train a CNN, and it would be hard to explain. Ye et al. (2022) did this with thousands of scans. HU features reach 0.94 AUC and are interpretable.

**Q: Isn't the bone-suppression result cheating?**
It's labelled post-hoc because I designed it after seeing errors on the same data. I report it separately, with the caveat that its gain is within noise. A fair test needs new data.

**Q: What would you do next?**
Segment the aorta and measure its HU directly; use 3-D series; test on another TCIA collection; add contrast-phase labels.

---

## 4. How to modify things safely

| Want to… | Change | Re-run from |
|---|---|---|
| Use a different target | `TARGET` in Part 1.4 | Part 1.4 |
| Try other HU bands | `BANDS` and 130/400 in `image_features` | Part 4 |
| Faster run | `N_REPEATS` (Part 0), `NESTED_REPEATS`, `N_PERM` | the changed part |
| Add a model | an entry in `make_models()` | Part 7 |
| Add a feature | a new `f['name'] = ...` line in `image_features` | Part 4 |
