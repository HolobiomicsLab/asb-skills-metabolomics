---
name: reference-material-bias-assessment
description: Use when a quantification pipeline has been applied to a certified reference
  material (e.g., NIST SRM-1950) with known reference concentrations and you must
  report per-analyte percent bias and mean bias to validate the method.
when_to_use_negative:
- The sample matrix or analyte list has no certified reference values, so percent
  bias versus reference concentrations cannot be computed
- The goal is peak annotation or integration fidelity itself (assessed via median
  annotation accuracy >99.9% and intermediate precision CV) rather than quantification
  accuracy versus a reference material
- Input is raw unprocessed data — the reference material must first be quantified
  through the full preprocessing and calibration workflow before bias can be assessed
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: PeakMeister
  role: R-based tool that preprocesses targeted full-scan MSI-CE-MS mzML data (migration
    index prediction, peak annotation, peak integration) to produce the peak areas
    converted to concentrations for bias assessment
  repo: https://github.com/PBM-Group/PeakMeister
- name: R
  role: Statistical environment in which PeakMeister is written and executed and in
    which bias calculations and CSV output are performed
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/acs.analchem.4c03513
    title: A Software Tool for Rapid and Automated Preprocessing of Large-Scale Serum
      Metabolomic Data by Multisegment Injection-Capillary Electrophoresis-Mass Spectrometry
  source_content_hash: sha256:4032fae41470b238aed3aba602cb6f7a34501dfbc3bc305b634550d70c16c677
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
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8666666666666667
n_claims: 15
---

# reference-material-bias-assessment

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Quantifies per-metabolite and mean percent bias of a preprocessing/quantification workflow against a certified reference material (NIST SRM-1950 plasma), as used to validate PeakMeister-processed MSI-CE-MS metabolite concentrations.

## When to use

Use when a quantification pipeline has been applied to a certified reference material (e.g., NIST SRM-1950) with known reference concentrations and you must report per-analyte percent bias and mean bias to validate the method.

## When NOT to use

- The sample matrix or analyte list has no certified reference values, so percent bias versus reference concentrations cannot be computed
- The goal is peak annotation or integration fidelity itself (assessed via median annotation accuracy >99.9% and intermediate precision CV) rather than quantification accuracy versus a reference material
- Input is raw unprocessed data — the reference material must first be quantified through the full preprocessing and calibration workflow before bias can be assessed

## Inputs

- NIST SRM-1950 mzML runs processed by the workflow (PeakMeister outputs: migration index table, peak migration time table, peak area table)
- NIST-reported reference concentrations for the shared metabolites
- Deposited calibration-curve data (Table S1) for converting peak areas to concentrations

## Outputs

- nist_srm1950_bias_table.csv with per-metabolite percent bias and mean bias across the 16 metabolites

## How to apply

First, run the quantification workflow (here, PeakMeister preprocessing of mzML runs from the 'mzML Files' folder parameterized via 'Mass List and Parameters.xlsx', using migration indexes from internal-standard 'Reference' peaks with extraction.window.ppm ≥ ~30-35, a migration.window.seconds search window, and snr.threshold below which peaks are recorded as '<LOD') to obtain peak areas for metabolites shared with the reference material. Convert peak areas to concentrations using deposited calibration-curve data (Table S1). For each of the 16 metabolites in common with NIST SRM-1950, compute percent bias = (measured concentration − NIST reference value) / NIST reference value × 100, then compute the mean bias across all metabolites and write the summary to nist_srm1950_bias_table.csv. Interpret bias relative to the expected validation envelope: mean bias ≈ −1.3% with per-metabolite bias within −28% (alanine) to +25% (cystine); critically, compare against a parallel manual-processing bias assessment so that shared bias is not attributed to the automated method.

## Related tools

- **PeakMeister** (R-based tool that preprocesses targeted full-scan MSI-CE-MS mzML data (migration index prediction, peak annotation, peak integration) to produce the peak areas converted to concentrations for bias assessment) — https://github.com/PBM-Group/PeakMeister
- **R** (Statistical environment in which PeakMeister is written and executed and in which bias calculations and CSV output are performed)

## Examples

```
Rscript code  # open the PeakMeister project R script 'code' after filling 'Mass List and Parameters.xlsx' and placing NIST SRM-1950 mzML files in 'mzML Files'; then compute bias vs NIST reference values into nist_srm1950_bias_table.csv
```

## Evaluation signals

- Mean bias across the 16 NIST SRM-1950 metabolites is −1.3% (accept if reproduced near this value)
- Per-metabolite biases fall within the reported range −28% (alanine) to +25% (cystine)
- Peak areas were converted to concentrations using the deposited Table S1 calibration-curve data, not ad hoc factors
- Bias of the automated (PeakMeister) path is similar to bias from manual preprocessing by an experienced analyst, so bias is not attributable to the preprocessing method
- Peaks below snr.threshold are correctly recorded as '<LOD' and excluded from bias computation

## Limitations

- Bias reflects the whole quantification chain (preprocessing plus calibration), not preprocessing alone — similar bias occurred with manual processing, so it was not attributed to the preprocessing method
- Only metabolites shared with the reference material (16 of the serum metabolite panel) can be bias-assessed
- Low abundance metabolites with signals near the method detection limit (recorded as '<LOD') and certain late migrating metabolites lacking flanking or stable-isotope internal standards are prone to peak misannotation, which can propagate into bias estimates

## Evidence

- [full_text] per-metabolite bias range from task card: [evidence span withheld — anchor status: not_found]
