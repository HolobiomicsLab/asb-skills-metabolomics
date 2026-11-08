# suspect-screening-exposomics-workflow

**Status:** published as an **outline** — the structure is validated, the execution is not.
**Kind:** composite-workflow (P1 canonical set).
**Bound by:** perspicacite-semantic (text-embedding-3-large retrieval + EDAM-aware primary selection).

> Automatic grading of `workflow.yaml` (`asb solve-workflow`, checkpoint mode) is **not part of this release**: no released ASB version loads these files. Follow the stages yourself. The stage structure is validated by `validate_workflows.py` through `release_gate.py`.

## Stages

1. **preprocess** — raw HRMS -> feature table + MS2 export  →  `lcms-feature-detection-and-quantification`, `lc-msms-feature-detection`, `lc-ms-feature-detection--e300bd20a9eef616`, `lc-ms-feature-quantification-processing`
2. **suspect_match** — features -> suspect-list matches by exact mass / RT / MS2  →  `accurate-mass-suspect-screening`, `retention-time-shift-and-noise-thresholding`, `msms-fragment-screening`, `mass-signal-cross-referencing-against-in-silico-predictions`
3. **in_silico_fragment** — suspect hits -> in-silico fragmentation structure ranking (MetFrag / SIRIUS)  →  `candidate-structure-retrieval-from-pubchem-chemspider`, `spectra-candidate-ranking-evaluation`, `in-silico-fragmentation-spectrum-prediction`, `structure-candidate-ranking-comparison`
4. **confidence** — assign identification confidence levels (Schymanski 1-5) to hits  →  `schymanski-identification-level-assignment`, `ms-annotation-confidence-level-assignment`, `compound-annotation-confidence-estimation`, `annotation-confidence-level-mapping`
5. **report** — consolidate suspect hits + structures + confidence into an annotated table  →  `csv-table-consolidation`, `gcms-hit-table-filtering`, `consensus-spectrum-candidate-ranking`, `compound-decontamination-filtering`

`derived_from_workflows` in the frontmatter is a provenance record — the ASB per-paper workflows whose structure corroborated this pipeline. No ablation experiment consuming it is released.
