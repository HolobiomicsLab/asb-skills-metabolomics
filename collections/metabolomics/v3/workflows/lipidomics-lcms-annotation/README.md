# lipidomics-lcms-annotation-workflow

**Status:** published as an **outline** — the structure is validated, the execution is not.
**Kind:** composite-workflow (P1 canonical set).
**Bound by:** perspicacite-semantic (text-embedding-3-large retrieval + EDAM-aware primary selection).

> Automatic grading of `workflow.yaml` (`asb solve-workflow`, checkpoint mode) is **not part of this release**: no released ASB version loads these files. Follow the stages yourself. The stage structure is validated by `validate_workflows.py` through `release_gate.py`.

## Stages

1. **preprocess** — raw lipidomics mzML -> aligned feature table + MS/MS export  →  `snakemake-workflow-rule-execution`, `targeted-lipidomics-transition-list-processing`, `lipidomics-peak-table-filtering`, `lipidomics-batch-effect-correction`, `mass-spectra-preprocessing`
2. **normalize** — normalize + batch-correct the lipid feature table  →  `combat-batch-effect-correction`, `lc-ms-feature-table-batch-correction`, `qc-sample-based-feature-normalization`, `batch-integration-of-ms-omics-data`
3. **lipid_identification** — identify lipids (class + species) from MS/MS fragmentation  →  `lipidblast-in-silico-spectral-matching`, `lipid-fragment-library-matching`, `rule-based-lipid-identification`, `ms2-ms3-fragment-peak-annotation`
4. **rule_validation** — validate lipid annotations by adduct / retention-time / class rules  →  `lipid-feature-qc-filtering`, `lipid-class-adduct-filtering`, `lipidomics-qc-filtering`, `adduct-aware-lipid-library-querying`
5. **statistics** — differential lipid analysis between sample groups  →  `between-group-fold-change-analysis`, `lipidomics-multivariate-statistical-analysis`, `metabolomics-feature-differential-analysis`, `scms-lipidomics-fold-change-analysis`
6. **fusion** — consolidate lipid annotations + stats into one master table  →  `lipidomics-annotation-appending`, `multi-dataset-annotation-aggregation`, `feature-table-assembly`, `cross-software-annotation-table-reconciliation`

`derived_from_workflows` in the frontmatter is a provenance record — the ASB per-paper workflows whose structure corroborated this pipeline. No ablation experiment consuming it is released.
