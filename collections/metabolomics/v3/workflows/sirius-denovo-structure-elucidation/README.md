# sirius-denovo-structure-elucidation-workflow

**Status:** published as an **outline** — the structure is validated, the execution is not.
**Kind:** composite-workflow (P1 canonical set).
**Bound by:** perspicacite-semantic (text-embedding-3-large retrieval + EDAM-aware primary selection).

> Automatic grading of `workflow.yaml` (`asb solve-workflow`, checkpoint mode) is **not part of this release**: no released ASB version loads these files. Follow the stages yourself. The stage structure is validated by `validate_workflows.py` through `release_gate.py`.

## Stages

1. **formula** — molecular formula determination (SIRIUS + ZODIAC)  →  `molecular-formula-reranking-zodiac`, `molecular-formula-determination-isotope-fragmentation`, `molecular-formula-gibbs-sampling`, `isotope-pattern-based-formula-ranking`
2. **custom_db** (optional) — (optional) build a custom structure database for the search space  →  `cheminformatics-structure-standardization`, `cross-dataset-molecule-deduplication`, `structure-database-size-validation`, `structural-descriptor-calculation`
3. **structure** — structure prediction (CSI:FingerID + COSMIC)  →  `fingerprint-based-candidate-re-ranking`, `csi-fingerid-fingerprint-prediction`, `structure-candidate-ranking-comparison`, `confidence-score-fdr-estimation`
4. **compound_class** — compound class prediction (CANOPUS / NPClassifier)  →  `compound-class-prediction-canopus`, `classyfire-npclassifier-ontology-mapping`, `classyfire-superclass-frequency-tabulation`, `classyfire-superclass-annotation-analysis`
5. **confidence_filter** — filter annotations by ZODIAC / COSMIC confidence  →  `zodiac-posterior-score-threshold-analysis`, `annotation-accuracy-benchmarking`, `mass-spectrometry-annotation-file-parsing`

`derived_from_workflows` in the frontmatter is a provenance record — the ASB per-paper workflows whose structure corroborated this pipeline. No ablation experiment consuming it is released.
