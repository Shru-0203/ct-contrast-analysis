# References (verified)

Each entry below was checked against the publisher, journal or archive page on 10 Oct 2026. Before citing one in your report, **open it and read at least the abstract** — cite only what you have actually used. Format shown is IEEE-style; switch to whatever your department requires.

## Dataset and data source

[1] K. Mader, "CT Medical Images: CT images from cancer imaging archive with contrast and patient age," Kaggle dataset. Available: https://www.kaggle.com/datasets/kmader/siim-medical-images

[2] B. Albertina, M. Watson, C. Holback, R. Jarosz, S. Kirk, Y. Lee, *et al.*, "Radiology Data from The Cancer Genome Atlas Lung Adenocarcinoma [TCGA-LUAD] collection," The Cancer Imaging Archive, 2016. doi: 10.7937/K9/TCIA.2016.JGNIHEP5
*TCIA lists this collection under the CC BY 3.0 licence.*

[3] K. Clark, B. Vendt, K. Smith, J. Freymann, J. Kirby, P. Koppel, S. Moore, S. Phillips, D. Maffitt, M. Pringle, L. Tarbox, F. Prior, "The Cancer Imaging Archive (TCIA): Maintaining and Operating a Public Information Repository," *Journal of Digital Imaging*, vol. 26, no. 6, pp. 1045–1057, 2013.
*TCIA asks every user of its data to cite this paper.*

## Related work (problem significance)

[4] Z. Ye, J. M. Qian, A. Hosny, R. Zeleznik, D. Plana, J. Likitlersuang, Z. Zhang, R. H. Mak, H. J. W. L. Aerts, B. H. Kann, "Deep Learning-based Detection of Intravenous Contrast Enhancement on CT Scans," *Radiology: Artificial Intelligence*, vol. 4, no. 3, e210285, 2022. doi: 10.1148/ryai.210285
*Closest related work: CNNs (EfficientNetB4 best) detect IV contrast on head-and-neck and chest CT because DICOM contrast fields are often missing or wrong. Trained on a far larger dataset than ours — useful to contrast "deep learning on thousands of scans" with "interpretable HU features on ~100 slices".*

## Methods

[5] R. M. Haralick, K. Shanmugam, I. Dinstein, "Textural Features for Image Classification," *IEEE Transactions on Systems, Man, and Cybernetics*, vol. SMC-3, no. 6, pp. 610–621, 1973. doi: 10.1109/TSMC.1973.4309314
*Origin of the GLCM texture features used in Part 4.*

[6] F. Pedregosa, G. Varoquaux, A. Gramfort, V. Michel, B. Thirion, O. Grisel, *et al.*, "Scikit-learn: Machine Learning in Python," *Journal of Machine Learning Research*, vol. 12, pp. 2825–2830, 2011.

[7] S. Kaufman, S. Rosset, C. Perlich, "Leakage in Data Mining: Formulation, Detection, and Avoidance," *ACM Transactions on Knowledge Discovery from Data*, vol. 6, no. 4, 2012. doi: 10.1145/2382577.2382579
*Background for the leakage audit (Part 2).*

[8] S. Varma, R. Simon, "Bias in error estimation when using cross-validation for model selection," *BMC Bioinformatics*, vol. 7, art. 91, 2006. doi: 10.1186/1471-2105-7-91
*Why tuning must be nested inside CV (Part 8.1).*

[9] M. Ojala, G. C. Garriga, "Permutation Tests for Studying Classifier Performance," *Journal of Machine Learning Research*, vol. 11, pp. 1833–1863, 2010.
*Basis of the label-permutation test (Part 8.2).*

[10] R. Geirhos, J.-H. Jacobsen, C. Michaelis, R. Zemel, W. Brendel, M. Bethge, F. A. Wichmann, "Shortcut learning in deep neural networks," *Nature Machine Intelligence*, vol. 2, no. 11, pp. 665–673, 2020. doi: 10.1038/s42256-020-00257-z
*Concept behind the acquisition-settings shortcut check (Part 7.4).*

## Software

[11] pydicom contributors, *pydicom* (Python package for reading DICOM files). Available: https://github.com/pydicom/pydicom — cite the version printed by the notebook's version cell.

## Not yet sourced — find these yourself

- A radiology textbook or review giving **typical Hounsfield-unit ranges** (air, fat, soft tissue, unenhanced vs contrast-enhanced blood, bone). The thresholds in Part 4 (−400, −190, −30, 130, 400 HU) follow standard CT ranges, but your report should cite a source you have read for them. Your library's radiology/medical-physics textbooks are the best place.
