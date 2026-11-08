# untargeted-lcmsms-annotation-workflow

**Status:** published as an **outline** — the structure is validated, the execution is not.
**Kind:** composite-workflow (P1 canonical set).
**Bound by:** perspicacite-semantic (text-embedding-3-large retrieval + EDAM-aware primary selection).

> Automatic grading of `workflow.yaml` (`asb solve-workflow`, checkpoint mode) is **not part of this release**: no released ASB version loads these files. Follow the stages yourself. The stage structure is validated by `validate_workflows.py` through `release_gate.py`.

## Stages

1. **preprocess** — raw mzML -> aligned feature table + MS2 exports (GNPS-FBMN mgf + SIRIUS mgf)  →  `lc-ms-peak-detection-and-feature-alignment`, `lcms-peak-detection-preprocessing`, `untargeted-metabolomics-preprocessing`, `lc-msms-feature-detection-and-alignment`
2. **qc_filter** (optional) — (optional) remove blank / background / low-quality features before annotation  →  `lcms-feature-quality-filtering`, `blank-subtraction-quality-filtering`, `blank-vs-qc-intensity-ratio-filtering`, `blank-sample-removal-qc`, `blank-filtering-of-ms-features`
3. **network** — MS2 spectra -> molecular family graph (modified cosine; GNPS-style components)  →  `feature-based-molecular-networking--a1d4da5bf30d5f86`, `msms-spectral-similarity-molecular-networking`, `feature-based-molecular-networking--e34d752ede072f7b`, `gnps-molecular-networking-workflow`
4. **library_match** — MS2 spectra -> spectral library annotations (cosine match to reference libraries)  →  `spectral-library-matching--c9ea083859d5db3b`, `mass-spectral-library-matching--d96cdbebfa644977`, `spectral-library-matching--c3bacc8572caec82`, `spectral-library-matching--599e7745b946f0de`
5. **sirius** — MS2 spectra -> formula + structure + class (SIRIUS / CSI:FingerID / CANOPUS)  →  `batch-ms2-annotation-automation`, `msms-spectra-sirius-csi-fingerid-annotation`, `csi-fingerid-structure-annotation`, `insilico-formula-annotation-sirius`
6. **fusion** — consolidate networking + library + SIRIUS into one master table, with optional taxonomy re-weighting  →  `multi-tool-annotation-table-construction`, `metabolite-annotation-mapping`, `metabolomics-feature-table-processing`, `mass-spectral-library-matching-cleanup`

`derived_from_workflows` in the frontmatter is a provenance record — the ASB per-paper workflows whose structure corroborated this pipeline. No ablation experiment consuming it is released.
