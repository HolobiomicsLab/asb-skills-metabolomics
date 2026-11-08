# in-silico-biotransformation-prediction-workflow

**Status:** published as an **outline** — the structure is validated, the execution is not.
**Kind:** composite-workflow (P1 canonical set).
**Bound by:** perspicacite-semantic (text-embedding-3-large retrieval + EDAM-aware primary selection).

> Automatic grading of `workflow.yaml` (`asb solve-workflow`, checkpoint mode) is **not part of this release**: no released ASB version loads these files. Follow the stages yourself. The stage structure is validated by `validate_workflows.py` through `release_gate.py`.

## Stages

1. **preprocess** — raw mzML -> feature table + MS2 export  →  `lcms-feature-detection-and-quantification`, `lc-ms-peak-detection-and-feature-alignment`, `untargeted-metabolomics-feature-extraction`, `lc-ms-feature-detection--512a653b5701a1cd`
2. **biotransformation_prediction** — parent structure -> predicted metabolite/transformation-product structures (rule-based expansion)  →  `cyp450-phasei-phaseii-biotransformation-rule-application`, `biotransformer-batch-metabolite-prediction`, `biotransformer-one-step-transformation-recapitulation`, `gut-microbial-biotransformation-prediction`
3. **candidate_filtering** — predicted candidates -> mass-plausible candidates (filter against the experimental peak list)  →  `metabolite-mass-filtering-by-exact-mass-tolerance`, `in-silico-biotransformation-prediction`, `candidate-mass-tolerance-filtering`, `metabolite-mass-accuracy-cross-referencing`
4. **ms_matching** — mass-plausible candidates -> MS/MS-matched and ranked biotransformation products  →  `candidate-molecule-fragmentation-scoring`, `in-silico-fragmentation-scoring--8f98c1a8185e1084`, `candidate-structure-retrieval-from-pubchem-chemspider`, `tandem-mass-spectra-candidate-ranking`
5. **report** — consolidate parent structure, predicted candidates, and MS/MS-matched hits into a biotransformation-product annotation table  →  `biotransformation-reaction-rule-matching`, `feature-table-curation`, `compound-annotation-table-generation`, `differential-feature-cross-referencing`

`derived_from_workflows` in the frontmatter is a provenance record — the ASB per-paper workflows whose structure corroborated this pipeline. No ablation experiment consuming it is released.
