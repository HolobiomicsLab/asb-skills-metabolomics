# gcms-deconvolution-identification-workflow

**Status:** published as an **outline** — the structure is validated, the execution is not.
**Kind:** composite-workflow (P1 canonical set).
**Bound by:** perspicacite-semantic (text-embedding-3-large retrieval + EDAM-aware primary selection).

> Automatic grading of `workflow.yaml` (`asb solve-workflow`, checkpoint mode) is **not part of this release**: no released ASB version loads these files. Follow the stages yourself. The stage structure is validated by `validate_workflows.py` through `release_gate.py`.

## Stages

1. **deconvolution** — GC-MS EI spectral deconvolution + peak detection  →  `gc-ms-spectral-deconvolution`, `gcms-spectral-deconvolution`, `dda-spectra-deconvolution`, `eic-peak-shape-correlation-deconvolution`
2. **ei_library_match** — identify compounds by EI spectral library matching  →  `gcms-spectral-library-matching`, `ei-mass-spectrum-similarity-scoring`, `molecular-fingerprint-prediction-from-spectra`, `ei-mass-spectrum-library-matching`
3. **retention_index** — retention index calibration + RI-filtered identifications  →  `retention-index-based-peak-refinement`, `retention-index-constrained-compound-identification`, `nalkylpyridinium-sulfonate-ri-calibration`, `retention-index-database-matching`
4. **statistics** — differential GC-MS feature analysis between groups  →  `metabolomics-univariate-group-comparison`, `univariate-differential-feature-testing`, `cell-type-differential-abundance-testing`, `metabolomics-feature-statistical-testing`
5. **fusion** — consolidate GC-MS identifications + stats into a master table  →  `gcms-quantification-table-generation`, `gcms-deconvolution-feature-scoring`, `gc-ms-quantitative-table-matching`, `cross-sample-feature-grouping`

`derived_from_workflows` in the frontmatter is a provenance record — the ASB per-paper workflows whose structure corroborated this pipeline. No ablation experiment consuming it is released.
