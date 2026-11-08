# masst-repository-scale-search-workflow

**Status:** published as an **outline** — the structure is validated, the execution is not.
**Kind:** composite-workflow (P1 canonical set).
**Bound by:** perspicacite-semantic (text-embedding-3-large retrieval + EDAM-aware primary selection).

> Automatic grading of `workflow.yaml` (`asb solve-workflow`, checkpoint mode) is **not part of this release**: no released ASB version loads these files. Follow the stages yourself. The stage structure is validated by `validate_workflows.py` through `release_gate.py`.

## Stages

1. **spectrum_prep** — prepare a query MS/MS spectrum / USI for search  →  `spectrum-usi-retrieval-from-online-repositories`, `universal-spectrum-identifier-construction`, `massive-dataset-spectrum-retrieval`, `usi-spectrum-searching`
2. **masst_search** — repository-scale spectral search (fastMASST)  →  `fast-search-api-query-execution`, `msms-spectral-batch-search-against-public-repositories`, `mass-spectral-analog-search`, `mass-spectrometry-batch-search`, `msms-spectral-search`
3. **specialized_masst** (optional) — (optional) ecological context via microbe/plant/food MASST  →  `mass-spectrometry-library-matching--f8a5aa74ab04f373`, `compound-to-taxon-mapping`
4. **cooccurrence** — co-occurrence / reverse-metabolomics interpretation  →  `human-sample-context-stratification-of-msms-matches`, `masst-job-result-parsing`, `masst-match-table-parsing`, `massive-public-dataset-spectrum-search-retrieval`

`derived_from_workflows` in the frontmatter is a provenance record — the ASB per-paper workflows whose structure corroborated this pipeline. No ablation experiment consuming it is released.
