# statistics-biomarker-discovery-workflow

**Status:** published as an **outline** — the structure is validated, the execution is not.
**Kind:** composite-workflow (P1 canonical set).
**Bound by:** perspicacite-semantic (text-embedding-3-large retrieval + EDAM-aware primary selection).

> Automatic grading of `workflow.yaml` (`asb solve-workflow`, checkpoint mode) is **not part of this release**: no released ASB version loads these files. Follow the stages yourself. The stage structure is validated by `validate_workflows.py` through `release_gate.py`.

## Stages

1. **clean_normalize** — clean + normalize the feature table  →  `svr-based-intensity-normalization`, `lc-ms-feature-table-cleaning`, `metabolomics-feature-filtering-and-imputation`, `lipidomic-feature-table-preprocessing`
2. **multivariate** — multivariate analysis (PCA / PLS-DA / OPLS-DA, VIP)  →  `pls-da-multivariate-modeling`, `plsda-supervised-classification-metabolomics`, `pca-dimension-reduction-metabolomics`, `metabolomics-principal-component-analysis`
3. **differential** — differential feature analysis (univariate, volcano, FDR)  →  `pairwise-group-comparison-statistics`, `metabolite-univariate-statistical-testing`, `metabolomics-feature-differential-analysis`, `cell-type-differential-abundance-testing`, `metabolomics-peak-differential-analysis`
4. **enrichment_pathway** — enrichment + pathway / functional analysis  →  `mummichog-pathway-enrichment-analysis`, `pathway-enrichment-analysis-fishers-exact-test`, `pathway-enrichment-analysis`, `mummichog-pathway-activity-prediction`
5. **biomarker** — biomarker selection + ROC / importance  →  `feature-importance-rank-scoring`, `random-forest-classifier-training--728f93a1a0de83da`, `metabolomics-feature-importance-ranking`, `multi-feature-set-model-aggregation`

`derived_from_workflows` in the frontmatter is a provenance record — the ASB per-paper workflows whose structure corroborated this pipeline. No ablation experiment consuming it is released.
