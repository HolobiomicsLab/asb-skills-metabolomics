# targeted-lcms-quantification-workflow

**Status:** published as an **outline** — the structure is validated, the execution is not.
**Kind:** composite-workflow (P1 canonical set).
**Bound by:** perspicacite-semantic (text-embedding-3-large retrieval + EDAM-aware primary selection).

> Automatic grading of `workflow.yaml` (`asb solve-workflow`, checkpoint mode) is **not part of this release**: no released ASB version loads these files. Follow the stages yourself. The stage structure is validated by `validate_workflows.py` through `release_gate.py`.

## Stages

1. **integrate** — raw targeted LC-MS -> integrated peak areas for target transitions  →  `lcms-targeted-peak-integration`, `lcms-peak-integration-evaluation`, `extracted-ion-chromatogram-peak-integration`, `srm-hrms-targeted-chromatogram-extraction`
2. **calibrate** — calibrant standards -> calibration curves with internal-standard normalization  →  `calibration-curve-regression-fitting--d8bf18a2234ca88f`, `lc-ms-quantification-linear-quadratic-modeling`, `calibration-curve-regression-fitting--c78126913238471b`, `calibration-curve-regression-model-selection`
3. **quantify** — apply calibration -> absolute / relative concentrations per sample  →  `external-calibration-conversion`, `calibration-curve-based-quantification`, `metabolite-calibration-curve-quantification`
4. **qc** (optional) — (optional) batch QC — response drift, QC-sample RSD, outlier flagging  →  `batch-correction-effect-assessment`, `lc-ms-batch-correction-with-pooled-qc-samples`, `targeted-ms-quantification-qc-evaluation`, `qc-sample-drift-evaluation`
5. **report** — consolidate concentrations + QC into a reportable quantification table  →  `targeted-mrm-data-handling`, `lc-ms-qc-rsd-calculation`, `qc-metrics-report-generation`, `qc-dashboard-metric-tabulation`, `targeted-quantification-concentration-calculation`

`derived_from_workflows` in the frontmatter is a provenance record — the ASB per-paper workflows whose structure corroborated this pipeline. No ablation experiment consuming it is released.
