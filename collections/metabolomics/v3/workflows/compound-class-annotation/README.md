# compound-class-annotation-workflow

**Status:** published as an **outline** — the structure is validated, the execution is not.
**Kind:** composite-workflow (P1 canonical set).
**Bound by:** perspicacite-semantic (text-embedding-3-large retrieval + EDAM-aware primary selection).

> Automatic grading of `workflow.yaml` (`asb solve-workflow`, checkpoint mode) is **not part of this release**: no released ASB version loads these files. Follow the stages yourself. The stage structure is validated by `validate_workflows.py` through `release_gate.py`.

## Stages

1. **preprocess** — raw mzML -> feature table + SIRIUS-flavour MS2 export  →  `lc-msms-feature-detection`, `mass-spectrometry-feature-detection`, `lc-ms-feature-quantification-processing`, `lc-ms-feature-detection--e300bd20a9eef616`
2. **formula** — MS2 spectra -> molecular formula (SIRIUS + ZODIAC re-ranking)  →  `molecular-formula-reranking-zodiac`, `molecular-formula-determination-isotope-fragmentation`, `isotope-pattern-based-formula-ranking`, `molecular-formula-gibbs-sampling`
3. **fingerprint** — formula + MS2 -> molecular fingerprint (CSI:FingerID)  →  `csi-fingerprint-prediction`, `substructure-fingerprint-prediction`, `molecular-fingerprint-prediction`, `csi-fingerid-fingerprint-prediction`
4. **classify** — fingerprint -> compound class (CANOPUS / NPClassifier: superclass/class/pathway)  →  `msms-spectra-canopus-class-prediction`, `classyfire-npclassifier-ontology-mapping`, `in-silico-class-prediction-interpretation`, `molecular-fingerprint-encoding--7b81bc09ce5106ca`
5. **consolidate** — consolidate formula + fingerprint + class into a class-annotated feature table  →  `mass-spectrometry-feature-table-processing`, `mass-spectrometry-feature-annotation-qc`, `insilico-ms2-annotation-sirius-canopus`, `batch-ms2-annotation-automation`

`derived_from_workflows` in the frontmatter is a provenance record — the ASB per-paper workflows whose structure corroborated this pipeline. No ablation experiment consuming it is released.
