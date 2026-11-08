# nmr-metabolomics-profiling-workflow

**Status:** published as an **outline** — the structure is validated, the execution is not.
**Kind:** composite-workflow (P1 canonical set).
**Bound by:** perspicacite-semantic (text-embedding-3-large retrieval + EDAM-aware primary selection).

> Automatic grading of `workflow.yaml` (`asb solve-workflow`, checkpoint mode) is **not part of this release**: no released ASB version loads these files. Follow the stages yourself. The stage structure is validated by `validate_workflows.py` through `release_gate.py`.

## Stages

1. **preprocess_nmr** — NMR spectral preprocessing (phase, baseline, referencing, binning)  →  `fid-to-spectrum-preprocessing-pipeline`, `nmr-preprocessing-pipeline-interfacing`, `metabolomics-nmr-data-processing`, `spectral-preprocessing-for-multivariate-analysis`
2. **identification** — identify metabolites by chemical shift matching  →  `spectral-library-matching--0c95a117d483839e`, `metabolite-database-interrogation`, `nmr-reference-spectrum-retrieval-from-hmdb`, `nmr-metabolite-identification`
3. **quantification** — quantify metabolites from NMR signals  →  `nmr-spectral-deconvolution`, `metabolite-peak-amplitude-quantification`, `metabolite-concentration-estimation-nmr`, `nmr-mixture-quantification`
4. **statistics** — differential analysis of NMR profiles (univariate; multivariate where a leaf exists)  →  `metabolomics-univariate-group-comparison`, `two-sample-t-test-biomarker-testing`, `multiple-testing-correction`, `benjamini-hochberg-multiple-testing-correction--167cbe6451059b92`

`derived_from_workflows` in the frontmatter is a provenance record — the ASB per-paper workflows whose structure corroborated this pipeline. No ablation experiment consuming it is released.
