---
name: peak-integration-accuracy-benchmarking
description: Use when an automated CE-MS/MSI-CE-MS peak annotation or integration
  pipeline must be validated, i.e., you have a ground-truth annotation/peak-area table
  or expert manual preprocessing results to compare against quantitatively.
when_to_use_negative:
- The data are untargeted feature-table outputs already aligned and annotated, not
  targeted mzML EIEs requiring reference-standard-based annotation.
- Sample matrices other than serum (urine, saliva, stool) are being benchmarked for
  the first time — the paper states PeakMeister is not yet validated for non-serum
  matrices, so serum benchmarks cannot be assumed transferable.
- Late-migrating metabolites lacking flanking or stable-isotope internal standards,
  or signals near the detection limit, are the sole targets — these are documented
  failure cases, so a high-accuracy claim cannot be trusted from this benchmark alone.
edam_operation: http://edamontology.org/operation_3215
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: PeakMeister
  role: R software executing MTI-based peak annotation and integration of MSI-CE-MS
    data being benchmarked
  repo: github.com/PBM-Group/PeakMeister
- name: MSI-CE-MS
  role: High-throughput separation platform whose multiplexed electropherograms are
    annotated and integrated
- name: mzML
  role: Open-source input file format consumed directly by PeakMeister
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1021/acs.analchem.4c03513
    title: A Software Tool for Rapid and Automated Preprocessing of Large-Scale Serum
      Metabolomic Data by Multisegment Injection-Capillary Electrophoresis-Mass Spectrometry
  source_content_hash: sha256:cc58988aa1a618f8d7a3d9bf3e3430a36b0813b9373c5ae5a01b199663dabde7
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/PBM-Group/PeakMeister
  tools:
  - MSI-CE-MS
  - PeakMeister
  - R
  - mzML
  edam_operation: http://edamontology.org/operation_3215
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
  techniques:
  - CE-MS
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8461538461538461
n_claims: 13
---

# peak-integration-accuracy-benchmarking

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmark automated peak annotation and integration of MSI-CE-MS metabolomic data against a ground-truth reference (manual expert preprocessing or simulated data with known peak positions/areas), using PeakMeister's reported serum performance (median annotation accuracy >99.9%, mean integration bias = -2.1%) as the reference bar.

## When to use

Use when an automated CE-MS/MSI-CE-MS peak annotation or integration pipeline must be validated, i.e., you have a ground-truth annotation/peak-area table or expert manual preprocessing results to compare against quantitatively.

## When NOT to use

- The data are untargeted feature-table outputs already aligned and annotated, not targeted mzML EIEs requiring reference-standard-based annotation.
- Sample matrices other than serum (urine, saliva, stool) are being benchmarked for the first time — the paper states PeakMeister is not yet validated for non-serum matrices, so serum benchmarks cannot be assumed transferable.
- Late-migrating metabolites lacking flanking or stable-isotope internal standards, or signals near the detection limit, are the sole targets — these are documented failure cases, so a high-accuracy claim cannot be trusted from this benchmark alone.

## Inputs

- mzML files of MSI-CE-MS runs placed in the 'mzML Files' folder
- 'Mass List and Parameters.xlsx' template (Sheet 1 analyte mass list, Sheet 2 reference internal standards, Sheet 3 lock-mass and injection parameters)
- Ground-truth annotation and peak-area table (from expert manual preprocessing or simulated data)

## Outputs

- PeakMeister migration-index / relative migration time table
- Migration-time table of observed and expected peak positions
- Peak-area table per analyte per injection
- Extracted ion electropherogram (EIE) plots for annotation review
- annotation_accuracy_mae.csv with annotation accuracy and peak-area MAE vs ground truth

## How to apply

Run the automated pipeline (e.g., PeakMeister in R on mzML files, configured via the 'Mass List and Parameters.xlsx' template with extraction.window.ppm >= ~30-35, migration.window.seconds, snr.threshold, and 11 reference internal standards with class='Reference') to produce its migration-index table, migration-time table, peak-area table, and extracted ion electropherogram plots. Compare predicted peak positions, annotations, and integrated areas against ground truth: compute per-analyte annotation accuracy and the mean absolute error (MAE) or mean bias of peak areas relative to manual expert preprocessing. Judge success against the paper's serum benchmarks — median annotation accuracy >99.9%, mean peak-integration bias of -2.1%, and median intermediate-precision CV = 16.0% — and verify that sample position assignment is consistent across multiplexed injections. Pay special attention to failure-prone cases the paper identifies: late-migrating metabolites lacking two flanking or a stable-isotope internal standard, and low-abundance metabolites with S/N near the method detection limit (recorded as '<LOD').

## Related tools

- **PeakMeister** (R software executing MTI-based peak annotation and integration of MSI-CE-MS data being benchmarked) — github.com/PBM-Group/PeakMeister
- **MSI-CE-MS** (High-throughput separation platform whose multiplexed electropherograms are annotated and integrated)
- **mzML** (Open-source input file format consumed directly by PeakMeister)

## Examples

```
Rscript code  # open the PeakMeister project 'code' script in R after filling 'Mass List and Parameters.xlsx' and placing mzML files in 'mzML Files/'
```

## Evaluation signals

- Median annotation accuracy exceeds 99.9%, matching the paper's serum reference performance.
- Mean peak-integration bias vs manual preprocessing is small (paper: -2.1%); MAE of peak areas vs ground truth is quantified and reported.
- Intermediate precision (median CV) is acceptable (paper benchmark: 16.0% across batches).
- Mutual agreement with expert manual annotation holds for NIST SRM-1950 plasma metabolites (paper: mean bias = -1.3% for 16 metabolites).
- Sample position assignment is consistent across all multiplexed injections, and EIE plots show correct peak annotation for every analyte.

## Limitations

- PeakMeister has not been validated for matrices other than serum (e.g., urine, saliva, stool).
- Late-migrating serum metabolites lacking two flanking or a stable-isotope internal standard are prone to peak misannotation.
- Low-abundance metabolites with signals near the method detection limit are prone to misannotation and are recorded as '<LOD' when S/N falls below snr.threshold.
- A user-friendly GUI and real-time (non-retrospective) preprocessing output are not yet implemented.

## Evidence
