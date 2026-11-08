# ion-mobility-4d-annotation-workflow

**Status:** published as an **outline** — the structure is validated, the execution is not.
**Kind:** composite-workflow (P1 canonical set).
**Bound by:** perspicacite-semantic (text-embedding-3-large retrieval + EDAM-aware primary selection).

> Automatic grading of `workflow.yaml` (`asb solve-workflow`, checkpoint mode) is **not part of this release**: no released ASB version loads these files. Follow the stages yourself. The stage structure is validated by `validate_workflows.py` through `release_gate.py`.

## Stages

1. **preprocess_4d** — 4D LC-IMS-MS/MS feature extraction (with CCS)  →  `ion-mobility-mass-spectrometry-data-processing`, `lc-ims-msms-feature-detection`, `ion-mobility-ms-data-processing`, `ion-mobility-separation-profile-matching`
2. **ccs_calibration** — collision cross section calibration + filtering  →  `collision-cross-section-calibration`, `ion-mobility-ccs-calibration`, `collision-cross-section-calculation`, `reduced-collision-cross-section-correction`
3. **ccs_library_match** — CCS-aware spectral / library annotation  →  `ccs-based-compound-identification`, `msms-longitudinal-fragment-profiling-scoring`
4. **networking** (optional) — (optional) molecular networking of IM-resolved features  →  `feature-based-molecular-networking--7b025a18b065ae1c`, `feature-based-molecular-networking--f72759146010fbc6`, `modified-cosine-similarity-networking`, `feature-based-molecular-networking--a1d4da5bf30d5f86`

`derived_from_workflows` in the frontmatter is a provenance record — the ASB per-paper workflows whose structure corroborated this pipeline. No ablation experiment consuming it is released.
