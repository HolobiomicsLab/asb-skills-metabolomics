---
name: targeted-lcms-quantification-workflow
description: 'Use when you have targeted LC-MS data for a defined panel of analytes
  and want absolute or relative concentrations — extract and integrate the target
  transitions/ion chromatograms, build calibration curves from standards with internal-standard
  normalization, apply them to samples, and QC the batch (response drift, QC-sample
  RSD) to a reportable quantification table.

  '
license: CC-BY-4.0
metadata:
  kind: composite-workflow
  techniques: []
  stage_count: 5
  member_skills:
  - lcms-targeted-peak-integration
  - lcms-peak-integration-evaluation
  - extracted-ion-chromatogram-peak-integration
  - srm-hrms-targeted-chromatogram-extraction
  - calibration-curve-regression-fitting--d8bf18a2234ca88f
  - lc-ms-quantification-linear-quadratic-modeling
  - calibration-curve-regression-fitting--c78126913238471b
  - calibration-curve-regression-model-selection
  - external-calibration-conversion
  - calibration-curve-based-quantification
  - metabolite-calibration-curve-quantification
  - batch-correction-effect-assessment
  - lc-ms-batch-correction-with-pooled-qc-samples
  - targeted-ms-quantification-qc-evaluation
  - qc-sample-drift-evaluation
  - targeted-mrm-data-handling
  - lc-ms-qc-rsd-calculation
  - qc-metrics-report-generation
  - qc-dashboard-metric-tabulation
  - targeted-quantification-concentration-calculation
  member_tools:
  - R
  - TARDIS
  - MultiQuant
  - MultiQuant 3.0.3
  - QuantyFey
  - R 4.2.0
  - Shiny
  - Shiny 1.10.0
  - renv 1.1.4
  - R-Shiny
  - mzQuality
  - mzQualityDashboard
  coverage_gaps: []
  derived_from_workflows:
  - coll_fbmn_stats_cq
  - coll_peakqc_cq
  bound_by: perspicacite-semantic
  collection: https://w3id.org/holobiomicslab/asb-skill/collection/metabolomics/v3
schema_version: 0.3.0
attribution:
  generator: AgenticScienceBuilder
  promoter: Louis-Félix Nothias
  sponsor: CNRS & Université Côte d'Azur
---

# Targeted LC-MS Quantification (calibration -> absolute concentrations)

## Summary

Targeted transitions in, a QC'd quantification table out: peak integration, calibration-curve fitting, internal-standard normalization, and batch QC.


## When to use

Use when you have targeted LC-MS data for a defined panel of analytes and want absolute or relative concentrations — extract and integrate the target transitions/ion chromatograms, build calibration curves from standards with internal-standard normalization, apply them to samples, and QC the batch (response drift, QC-sample RSD) to a reportable quantification table.


## When NOT to use

- The data is not this platform.
- You need a single atomic step, not the full pipeline (use the leaf skill directly via the router).

## Stages

### Stage 1 — integrate

**Goal:** raw targeted LC-MS -> integrated peak areas for target transitions

**EDAM operation:** http://edamontology.org/operation_3215

**Inputs:** mzML · **Outputs:** feature-table

**Candidate leaf skills:** `lcms-targeted-peak-integration` (primary), `lcms-peak-integration-evaluation`, `extracted-ion-chromatogram-peak-integration`, `srm-hrms-targeted-chromatogram-extraction`

**Tools (primary):** R, TARDIS

**Other candidate tools:** Panorama, ProteoWizard, Skyline, HMDB, METLIN, MassBank, ProteoWizard (pwiz)

**Grounding:** 2 KB(s); DOIs: 10.1021/acs.analchem.5c00567, 10.1021/acs.jproteome.9b00640

### Stage 2 — calibrate

**Goal:** calibrant standards -> calibration curves with internal-standard normalization

**EDAM operation:** http://edamontology.org/operation_3435

**Inputs:** feature-table · **Outputs:** tsv

**Candidate leaf skills:** `calibration-curve-regression-fitting--d8bf18a2234ca88f` (primary), `lc-ms-quantification-linear-quadratic-modeling`, `calibration-curve-regression-fitting--c78126913238471b`, `calibration-curve-regression-model-selection`

**Tools (primary):** MultiQuant, MultiQuant 3.0.3, QuantyFey, R 4.2.0, Shiny, Shiny 1.10.0, renv 1.1.4

**Other candidate tools:** GetFeatistics, R

**Grounding:** 2 KB(s); DOIs: 10.1016/j.aca.2025.344571, 10.1515/jib-2025-0047

### Stage 3 — quantify

**Goal:** apply calibration -> absolute / relative concentrations per sample

**EDAM operation:** http://edamontology.org/operation_3799

**Inputs:** feature-table, tsv · **Outputs:** tsv

**Candidate leaf skills:** `external-calibration-conversion` (primary), `calibration-curve-based-quantification`, `metabolite-calibration-curve-quantification`

**Tools (primary):** MultiQuant, MultiQuant 3.0.3, QuantyFey, R 4.2.0

**Other candidate tools:** GetFeatistics, R, ggplot, MSI-CE-MS, NIST SRM-1950, PeakMeister

**Grounding:** 3 KB(s); DOIs: 10.1016/j.aca.2025.344571, 10.1021/acs.analchem.4c03513, 10.1515/jib-2025-0047

### Stage 4 — qc  [OPTIONAL]

**Goal:** (optional) batch QC — response drift, QC-sample RSD, outlier flagging

**EDAM operation:** http://edamontology.org/operation_3435

**Inputs:** tsv · **Outputs:** tsv

**Candidate leaf skills:** `batch-correction-effect-assessment` (primary), `lc-ms-batch-correction-with-pooled-qc-samples`, `targeted-ms-quantification-qc-evaluation`, `qc-sample-drift-evaluation`

**Tools (primary):** R, R-Shiny, mzQuality, mzQualityDashboard

**Other candidate tools:** SCIEX OS 2.1.6.59781, QuantyFey, R 4.2.0, Python, TidyMS, scikit-learn

**Grounding:** 3 KB(s); DOIs: 10.1016/j.aca.2025.344571, 10.1021/jasms.5c00073, 10.3390/metabo10100416

### Stage 5 — report

**Goal:** consolidate concentrations + QC into a reportable quantification table

**EDAM operation:** http://edamontology.org/operation_3434

**Inputs:** tsv · **Outputs:** tsv

**Candidate leaf skills:** `targeted-mrm-data-handling` (primary), `lc-ms-qc-rsd-calculation`, `qc-metrics-report-generation`, `qc-dashboard-metric-tabulation`, `targeted-quantification-concentration-calculation`

**Tools (primary):** R, mzQuality, mzQualityDashboard

**Other candidate tools:** IonToolPack, PeakQC, MS-DIAL, MSConvert, Python, Rapid QC-MS, MultiQuant, QuantyFey, R 4.2.0, Shiny

**Grounding:** 4 KB(s); DOIs: 10.1016/j.aca.2025.344571, 10.1021/acs.analchem.4c00786, 10.1021/jasms.4c00146, 10.1021/jasms.5c00073

## Grounding

Each stage carries the `kb_slugs`/`dois` of the leaves it draws on. Ground any stage against its source paper with the collection's `/ground` command or `bin/perspicacite_kb_bind.py` (Perspicacité KB; serverless local-clone fallback).

## Verification contract

`workflow.yaml` declares the stage graph and its typed outputs; the final stage emits the master deliverable. Automatic grading of that graph (`asb solve-workflow`, checkpoint mode) is **not part of this release**: no released ASB version loads these files. Follow the stages as an outline — the structure is validated, the execution is not.

## Provenance

Generated by `compose_workflows.py` (semantic binding + EDAM-aware primary selection). `derived_from_workflows` lists the ASB per-paper workflows whose structure corroborated this pipeline; it is a provenance record, and no ablation experiment consuming it is released. Validated structurally by `validate_workflows.py` through `release_gate.py`: the collection is the hard-gated artefact and this workflows layer is additive.
