---
name: plsda-biomarker-selection
description: Use when after batch effect correction (e.g. WaveICA/WaveICA 2.0) of
  a metabolomics feature table with known class labels, you need to select discriminant
  metabolite peaks and quantify their predictive performance for the class contrast.
when_to_use_negative:
- Input is a raw LC–MS feature matrix that has not yet been batch-effect-corrected
  — apply WaveICA/WaveICA 2.0 first, since residual batch variance confounds PLS-DA
  selection
- Data lack class labels (e.g. QCS-only or single-class samples) — PLS-DA is a supervised
  discriminant method and cannot select discriminant biomarkers without a contrast
  like CRC vs CE
- You only need batch-effect removal quality assessment (QCS distances, PCCs) — those
  metrics do not require PLS-DA biomarker selection
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: MetaboAnalystR
  role: Metabolomics data analysis pipeline in which PLS-DA-based biomarker identification
    and evaluation are performed in R
- name: WaveICA_2.0
  role: Provides the batch-effect-corrected feature matrix and bundled Amide_data
    used as input to PLS-DA biomarker selection
  repo: https://github.com/dengkuistat/WaveICA_2.0
- name: R
  role: Implementation environment for PLS-DA, SVM cross-validation, and metric compilation
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1007/s11306-021-01839-7
    title: 'WaveICA 2.0: a novel batch effect removal method for untargeted metabolomics
      data without using batch information'
  source_content_hash: sha256:3243002e2f8d99b181775c76ed41e78df4c2a0ecc551e21bb2c5960735cc68a9
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/dengkuistat/WaveICA_2.0
  tools:
  - MetaboAnalystR
  - R
  - WaveICA_2.0
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# PLS-DA biomarker selection

## Summary

Identifies candidate metabolomic biomarkers from a batch-effect-corrected feature matrix using PLS-DA with the first three latent variables, selecting peaks with FDR < 0.05 and VIP > 1, then validating their predictive ability with a repeated five-fold cross-validated RBF SVM (AUC).

## When to use

Use when, after batch effect correction (e.g. WaveICA/WaveICA 2.0) of a metabolomics feature table with known class labels, you need to select discriminant metabolite peaks and quantify their predictive performance for the class contrast.

## When NOT to use

- Input is a raw LC–MS feature matrix that has not yet been batch-effect-corrected — apply WaveICA/WaveICA 2.0 first, since residual batch variance confounds PLS-DA selection
- Data lack class labels (e.g. QCS-only or single-class samples) — PLS-DA is a supervised discriminant method and cannot select discriminant biomarkers without a contrast like CRC vs CE
- You only need batch-effect removal quality assessment (QCS distances, PCCs) — those metrics do not require PLS-DA biomarker selection

## Inputs

- Batch-effect-corrected metabolomics feature matrix (samples × metabolite peaks, e.g. 642 samples × 6,461 peaks from Amide_data)
- Class labels for subjects (e.g. 497 colorectal cancer (CRC) vs 71 chronic enteritis (CE) patients)
- FDR correction settings and PLS-DA latent-variable count (3), VIP cutoff (> 1), FDR cutoff (< 0.05)

## Outputs

- List of selected biomarker peaks with FDR and VIP values (e.g. 172 peaks for WaveICA 2.0)
- Predictive performance metric: AUC from 5-fold cross-validated RBF SVM repeated 100 times (mean ± SD, e.g. 0.8831 ± 0.0170)
- Per-condition biomarker counts and AUCs compiled into results_table.csv for comparison (Table 1)

## How to apply

Standardize the corrected peak-intensity matrix (6,461 metabolite peaks from the Amide_data dataset) to mean 0 and variance 1, then fit PLS-DA using the first three latent variables on the class labels (e.g. CRC vs CE). Retain peaks meeting both thresholds: FDR-corrected P-value less than 0.05 and PLS-DA variable importance in projection (VIP) greater than 1; record the number of selected peaks (the paper reports 172 for WaveICA 2.0). Validate predictive ability by five-fold cross-validation of a radial-basis-function SVM repeated 100 times, recording mean ± SD AUC (reported as 0.8831 ± 0.0170 for WaveICA 2.0). Repeat the identical procedure on original and alternative corrected datasets so biomarker counts and AUCs are comparable across conditions, as compiled into the Table 1 results_table.csv.

## Related tools

- **MetaboAnalystR** (Metabolomics data analysis pipeline in which PLS-DA-based biomarker identification and evaluation are performed in R)
- **WaveICA_2.0** (Provides the batch-effect-corrected feature matrix and bundled Amide_data used as input to PLS-DA biomarker selection) — https://github.com/dengkuistat/WaveICA_2.0
- **R** (Implementation environment for PLS-DA, SVM cross-validation, and metric compilation)

## Examples

```
library(devtools); devtools::install_github("dengkuistat/WaveICA_2.0", host="https://api.github.com"); data(Amide_data); # PLS-DA on standardized corrected matrix, 3 latent variables, FDR<0.05 & VIP>1
```

## Evaluation signals

- Selected peak count falls in the reported range for Table 1 (e.g. WaveICA 2.0 selects 172 peaks at FDR < 0.05 and VIP > 1, vs 56 for QC-RLSC and 68 for QC-SVRC)
- Repeated 5-fold CV AUC of the RBF SVM reproduces the reported mean ± SD within a reasonable tolerance (WaveICA 2.0: 0.8831 ± 0.0170 for CRC vs CE)
- All retained peaks satisfy both thresholds simultaneously: FDR < 0.05 and VIP > 1
- Biomarker count and AUC are computed identically across original, WaveICA, and WaveICA 2.0 conditions so rows of results_table.csv are directly comparable
- PLS-DA uses exactly the first three latent variables on data standardized to mean 0 and variance 1 — deviations change peak counts and invalidate comparison

## Limitations

- Results depend on the upstream batch effect removal method: biomarker counts and AUCs differ across correction methods (172 peaks/0.8831 AUC for WaveICA 2.0 vs 68/0.8476 for QC-SVRC), so selection is not correction-agnostic
- The SVM cross-validation AUC is stochastic (repeated 100 times to estimate mean ± SD); a single run will not exactly reproduce reported values
- The reported dataset is a single unnamed plasma metabolomics study (CRC vs CE); thresholds (FDR 0.05, VIP 1, 3 latent variables) were tuned for that contrast and may need re-tuning elsewhere

## Evidence

- [methods] Biomarker selection thresholds and PLS-DA setup: [evidence span withheld — anchor status: not_found]
- [methods] SVM cross-validation evaluation protocol: [evidence span withheld — anchor status: not_found]
- [intro] MetaboAnalystR as the analysis pipeline: [evidence span withheld — anchor status: not_found]
