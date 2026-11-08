# pathway-functional-analysis-workflow

**Status:** published as an **outline** — the structure is validated, the execution is not.
**Kind:** composite-workflow (P1 canonical set).
**Bound by:** perspicacite-semantic (text-embedding-3-large retrieval + EDAM-aware primary selection).

> Automatic grading of `workflow.yaml` (`asb solve-workflow`, checkpoint mode) is **not part of this release**: no released ASB version loads these files. Follow the stages yourself. The stage structure is validated by `validate_workflows.py` through `release_gate.py`.

## Stages

1. **feature_prep** — prepare a ranked m/z feature list for functional analysis  →  `lcms-feature-table-filtering--14a2cde736d2dabd`, `metabolite-log2fc-ranking`, `metabolomics-feature-table-generation--f513762b0eaf782d`, `lc-msms-feature-quantification-table-transposition`
2. **mummichog** — functional analysis directly from m/z (mummichog)  →  `mummichog-network-activity-prediction`, `mummichog-pathway-enrichment-analysis`, `mummichog-pathway-activity-prediction`
3. **pathway_enrichment** — pathway + metabolite-set enrichment  →  `pathway-enrichment-analysis-fishers-exact-test`, `pathway-enrichment-analysis`, `metabolite-pathway-enrichment-analysis--b4a4fd3929413241`, `pathway-over-representation-analysis--2a80d81751b143d3`
4. **interpretation** — interpret + visualize enriched pathways  →  `metabolic-pathway-enrichment-visualization`, `kegg-pathway-enrichment-interpretation`, `metabolomics-pathway-enrichment-execution`, `pathway-impact-plot-visualization`

`derived_from_workflows` in the frontmatter is a provenance record — the ASB per-paper workflows whose structure corroborated this pipeline. No ablation experiment consuming it is released.
