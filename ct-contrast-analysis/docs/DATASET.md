# Dataset reference and approval status

> ⚠️ **Approval required.** The course requires a dataset from Dr. Prerna's Moodle list, or prior approval. This Kaggle dataset has **not** been confirmed as approved. Confirm it is on the list (or get written approval) **before** submitting work based on it.

| Item | Value |
|---|---|
| Name | CT Medical Images |
| Kaggle page | https://www.kaggle.com/datasets/kmader/siim-medical-images |
| Original source | TCGA-LUAD collection, The Cancer Imaging Archive (doi: 10.7937/K9/TCIA.2016.JGNIHEP5) |
| TCIA licence for the source collection | CC BY 3.0 (check the Kaggle page for the Kaggle copy's terms) |

## Verified contents (inspected 10 Oct 2026 from the Kaggle `archive.zip`, 262 MB)

| Item | Verified value |
|---|---|
| Files | `overview.csv`; `dicom_dir/` (100 `.dcm`, 51 MB); `tiff_images/` (100 `.tif`, 201 MB); `full_archive.npz` (187 MB) |
| Records | 100 rows in `overview.csv`, one per CT slice; all 512×512 |
| CSV columns | `Age`, `Contrast`, `ContrastTag`, `raw_input_path`, `id`, `tiff_name`, `dicom_name` (plus a saved pandas index) |
| Target | `Contrast` (True/False): 50 / 50 |
| Label origin | Matches the curator's folder name in `raw_input_path` (`Contrast` / `No contrast`) for all 100 rows |
| `ContrastTag` | Copy of the DICOM `ContrastBolusAgent` field — `NONE` for 50 no-contrast **and** 36 contrast slices; **not** the label |
| Patients | 54 unique `PatientID`s (TCGA-xx-xxxx); 1–6 slices each; **9 patients have slices in both classes** |
| Duplicates | 6 pairs of pixel-identical slices (same patient, same label) → 94 unique slices used |
| Missing values (CSV) | `ContrastTag`: 2; all other columns complete |
| Patient metadata usable | Age (CSV = DICOM for all 100), Sex (DICOM, 54 M / 46 F) — both 100% complete |
| Patient metadata too sparse | Weight 24%, Height 6%, Smoking status 9%, Ethnic group 6%, History 38%, Pregnancy status 42% present |
| Leaky fields | File names, `raw_input_path`, `ContrastTag`, DICOM contrast tags, and free-text `StudyDescription` / `ProtocolName` / `RequestedProcedureDescription` / `CodeMeaning` (e.g. "CT CHEST WITHOUT CONTR") |
| Scanners | Siemens 67, GE 33; 19 models; kVp 100–140; slice thickness 1–10 mm |
| TIFF images | HU copies of the DICOM pixels (max difference 1 HU, from rounding) — no extra information |
| `full_archive.npz` | Stores pickled Python objects → **not loaded** (unpickling downloaded files can run arbitrary code) |

## How to cite

Cite both the TCIA collection [2] and the TCIA paper [3] from `REFERENCES.md`, and the Kaggle page [1] as the place you downloaded it from.
