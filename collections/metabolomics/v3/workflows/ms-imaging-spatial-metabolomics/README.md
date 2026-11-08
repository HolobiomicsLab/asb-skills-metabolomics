# ms-imaging-spatial-metabolomics-workflow

**Status:** published as an **outline** — the structure is validated, the execution is not.
**Kind:** composite-workflow (P1 canonical set).
**Bound by:** perspicacite-semantic (text-embedding-3-large retrieval + EDAM-aware primary selection).

> Automatic grading of `workflow.yaml` (`asb solve-workflow`, checkpoint mode) is **not part of this release**: no released ASB version loads these files. Follow the stages yourself. The stage structure is validated by `validate_workflows.py` through `release_gate.py`.

## Stages

1. **preprocess_imaging** — load imzML, peak pick, align m/z across pixels  →  `imzml-spectral-data-processing`, `pixel-wise-peak-picking-of-imaging-spectra`, `mass-spectrometry-imaging-signal-preprocessing`, `imzml-maldimsi-data-preprocessing`
2. **spatial_annotation** — annotate ion images to metabolites with FDR control  →  `maldi-metabolite-annotation-via-metaspace`, `mass-signature-database-annotation`, `molecular-identifier-mapping-between-imaging-and-lcmsms`, `exact-mass-metabolite-annotation`
3. **segmentation** — spatial segmentation / clustering into regions  →  `spatial-shrunken-centroids-pixel-clustering`, `tissue-region-annotation-from-lipid-co-localisation`, `mz-peak-spatial-colocalisation-analysis`, `mass-spectrometry-image-segmentation`
4. **region_statistics** — compare metabolite intensities across spatial regions  →  `spatial-cluster-differential-metabolite-analysis`, `pseudo-bulk-differential-metabolite-expression`, `pseudo-bulk-pooling-of-spatial-metabolite-intensities`, `differential-metabolite-abundance-testing--8274b5843fb2ecb1`

`derived_from_workflows` in the frontmatter is a provenance record — the ASB per-paper workflows whose structure corroborated this pipeline. No ablation experiment consuming it is released.
