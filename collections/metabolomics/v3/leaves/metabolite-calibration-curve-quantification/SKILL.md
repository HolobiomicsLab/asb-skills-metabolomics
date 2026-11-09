---
name: metabolite-calibration-curve-quantification
description: Use when PeakMeister peak-area tables for targeted metabolites are available
  and absolute concentrations or accuracy-vs-reference metrics (e.g., NIST SRM-1950
  bias) must be computed.
when_to_use_negative:
- Input is already a concentration/feature table — calibration conversion and bias
  calculation would double-process the data
- Data are not targeted full-scan MSI-CE-MS mzML files with a defined targeted mass
  list (PeakMeister was designed for multiplexed MSI-CE-MS datasets)
- No deposited calibration curves or reference values exist, so peak areas cannot
  be converted to concentrations or compared to ground truth
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: PeakMeister
  role: R-based tool that annotates and integrates analyte peaks in MSI-CE-MS mzML
    files and outputs the peak-area tables the calibration curves are applied to
  repo: https://github.com/PBM-Group/PeakMeister
- name: R
  role: Statistical environment in which the PeakMeister preprocessing script and
    downstream bias computations are executed
  repo: https://github.com/PBM-Group/PeakMeister
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/acs.analchem.4c03513
    title: A Software Tool for Rapid and Automated Preprocessing of Large-Scale Serum
      Metabolomic Data by Multisegment Injection-Capillary Electrophoresis-Mass Spectrometry
  source_content_hash: sha256:f31759dbb0e0a4b1fd2e35f7a5ebbc689a228d1ad50f0c9d27a74b70fb3f55f7
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/PBM-Group/PeakMeister
  repo_url: https://github.com/PBM-Group/PeakMeister
  tools:
  - MSI-CE-MS
  - NIST SRM-1950
  - PeakMeister
  - R
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
  techniques:
  - CE-MS
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 14
---

# metabolite-calibration-curve-quantification

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Converts MSI-CE-MS metabolite peak areas into absolute concentrations using deposited calibration-curve data, and validates accuracy by computing per-metabolite percent bias against NIST SRM-1950 plasma reference values.

## When to use

Use when PeakMeister peak-area tables for targeted metabolites are available and absolute concentrations or accuracy-vs-reference metrics (e.g., NIST SRM-1950 bias) must be computed.

## When NOT to use

- Input is already a concentration/feature table — calibration conversion and bias calculation would double-process the data
- Data are not targeted full-scan MSI-CE-MS mzML files with a defined targeted mass list (PeakMeister was designed for multiplexed MSI-CE-MS datasets)
- No deposited calibration curves or reference values exist, so peak areas cannot be converted to concentrations or compared to ground truth

## Inputs

- mzML files of MSI-CE-MS runs (e.g., NIST SRM-1950 plasma samples) placed in the 'mzML Files' folder
- 'Mass List and Parameters.xlsx' template with analyte m/z values, extraction.window.ppm, migration.window.seconds, snr.threshold, and reference internal standards
- Deposited calibration-curve data (Table S1) mapping peak areas to concentrations
- NIST SRM-1950 reference concentration values for the 16 common metabolites

## Outputs

- Table of migration indexes / relative migration times
- Table of peak migration times
- Table of peak areas
- Extracted ion electropherogram plots for annotation/integration checks
- nist_srm1950_bias_table.csv with per-metabolite and mean percent bias

## How to apply

First generate peak areas by running the PeakMeister R script on mzML runs, having filled the 'Mass List and Parameters.xlsx' template with extraction.window.ppm set to at least ~30-35 ppm, a migration.window.seconds search window, and an snr.threshold below which peaks are recorded as '<LOD'. Reference peaks (class 'Reference') within min.mt.min/max.mt.min cutoffs supply migration indexes that predict analyte migration times for annotation, with peak.space.tolerance.percent gap checking. Next apply the deposited calibration-curve data (Table S1) to convert the 16 metabolite peak areas into concentrations. Finally compute per-metabolite percent bias of the PeakMeister-derived concentrations versus NIST SRM-1950 reference values, plus the mean bias across all 16 metabolites, and inspect the extracted ion electropherogram plots to confirm proper annotation and integration.

## Related tools

- **PeakMeister** (R-based tool that annotates and integrates analyte peaks in MSI-CE-MS mzML files and outputs the peak-area tables the calibration curves are applied to) — https://github.com/PBM-Group/PeakMeister
- **R** (Statistical environment in which the PeakMeister preprocessing script and downstream bias computations are executed) — https://github.com/PBM-Group/PeakMeister

## Examples

```
Rscript code
```

## Evaluation signals

- Mean bias of PeakMeister-derived concentrations versus NIST SRM-1950 reference values should be approximately -1.3% across the 16 metabolites
- Per-metabolite biases should fall within the reported range of -28% (alanine) to +25% (cystine); similar bias to manual processing indicates preprocessing is not the cause
- Peaks with S/N below snr.threshold must be recorded as '<LOD' rather than quantified
- Extracted ion electropherogram plots must show proper peak annotation and integration for each metabolite
- Peak integration accuracy should mirror the article's mean bias of -2.1% and annotation fidelity of median accuracy >99.9% relative to manual preprocessing

## Limitations

- Per-metabolite bias can be large for individual metabolites (up to -28% for alanine and +25% for cystine), even when mean bias is small
- Late-migrating metabolites lacking two flanking or a stable-isotope internal standard, and low-abundance metabolites near the detection limit, are prone to peak misannotation
- PeakMeister is validated for serum/plasma matrices only; urine, saliva, and stool matrices are not yet supported

## Evidence
