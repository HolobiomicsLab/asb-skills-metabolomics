# stable-isotope-tracing-fluxomics-workflow

**Status:** published as an **outline** — the structure is validated, the execution is not.
**Kind:** composite-workflow (P1 canonical set).
**Bound by:** perspicacite-semantic (text-embedding-3-large retrieval + EDAM-aware primary selection).

> Automatic grading of `workflow.yaml` (`asb solve-workflow`, checkpoint mode) is **not part of this release**: no released ASB version loads these files. Follow the stages yourself. The stage structure is validated by `validate_workflows.py` through `release_gate.py`.

## Stages

1. **preprocess** — raw labelled LC-MS -> aligned feature table (all isotopologues)  →  `isotopologue-peak-quantification`, `isotope-label-tracing-analysis`, `stable-isotope-lc-ms-annotation`, `retention-time-shift-correction-isotope-labelling`
2. **isotopologue_extract** — feature table -> per-metabolite isotopologue intensity distributions  →  `isotope-hypothesis-enumeration-from-mass-traces`, `isotope-adduct-ion-grouping`, `isotopologue-intensity-forward-calculation`
3. **natural_abundance_correction** — correct isotopologue distributions for natural isotope abundance  →  `stable-isotope-lcms-labeling-data-processing`, `isotopologue-abundance-normalization`, `isotopic-enrichment-calculation`, `stable-isotope-labeling-data-quantification`
4. **labelling_analysis** — corrected distributions -> mass-isotopomer distribution / fractional enrichment  →  `mass-distribution-vector-enrichment-calculation`, `isotopomer-distribution-comparison`
5. **report** — consolidate labelling / enrichment results into a tracing report table  →  `metabolomics-data-summarization`, `isotopologue-labelling-visualization-of-metabolic-flux`, `metabolomics-html-report-generation`

`derived_from_workflows` in the frontmatter is a provenance record — the ASB per-paper workflows whose structure corroborated this pipeline. No ablation experiment consuming it is released.
