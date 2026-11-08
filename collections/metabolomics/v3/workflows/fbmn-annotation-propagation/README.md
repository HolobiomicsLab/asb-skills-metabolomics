# feature-based-molecular-networking-and-propagation-workflow

**Status:** published as an **outline** — the structure is validated, the execution is not.
**Kind:** composite-workflow (P1 canonical set).
**Bound by:** perspicacite-semantic (text-embedding-3-large retrieval + EDAM-aware primary selection).

> Automatic grading of `workflow.yaml` (`asb solve-workflow`, checkpoint mode) is **not part of this release**: no released ASB version loads these files. Follow the stages yourself. The stage structure is validated by `validate_workflows.py` through `release_gate.py`.

## Stages

1. **preprocess** — raw mzML -> aligned feature table + MS2 exports (GNPS-FBMN mgf + SIRIUS mgf)  →  `lc-ms-peak-detection-and-feature-alignment`, `lcms-peak-detection-preprocessing`, `untargeted-metabolomics-preprocessing`, `lc-msms-feature-detection-and-alignment`
2. **network** — MS2 spectra -> molecular family graph (modified cosine; GNPS-style components)  →  `feature-based-molecular-networking--7b025a18b065ae1c`, `modified-cosine-similarity-networking`, `feature-based-molecular-networking--a1d4da5bf30d5f86`, `msms-spectral-similarity-molecular-networking`
3. **seed_annotate** — MS2 spectra -> seed spectral-library annotations to propagate from  →  `modified-cosine-spectral-matching`, `spectral-library-matching--c3bacc8572caec82`, `spectral-library-matching--599e7745b946f0de`, `spectral-library-matching--c9ea083859d5db3b`
4. **class_annotate** — MS2 spectra -> molecular formula + chemical class (SIRIUS / CANOPUS / NPClassifier)  →  `msms-spectra-canopus-class-prediction`, `canopus-chemical-class-prediction`, `consensus-classification-propagation`, `in-silico-class-prediction-interpretation`
5. **propagate** — spread seed annotations + chemical classes across molecular families  →  `molecular-network-node-annotation-mapping`, `chemical-class-propagation-onto-networks`, `substructure-annotation-of-molecular-families`, `molecular-network-annotation-propagation--3c622bace7b9313a`
6. **consolidate** — consolidate network family + seed + class + propagated annotations into one table  →  `ms-feature-annotation-table-generation`, `molecular-network-construction-ms2-spectra`, `mass-spectrometry-feature-annotation-qc`, `mass-spectrometry-molecular-networking`

`derived_from_workflows` in the frontmatter is a provenance record — the ASB per-paper workflows whose structure corroborated this pipeline. No ablation experiment consuming it is released.
