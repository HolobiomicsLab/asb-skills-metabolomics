---
name: bqc-regression-slope-r2-evaluation
description: Use when a large multi-batch MRM dataset contains batch-pooled QC (BQC)
  samples and you need to quantify whether batch-effect correction (e.g., QC-RLSC)
  removed injection-order drift.
when_to_use_negative:
- The dataset contains no QC or batch-pooled QC samples, so batch-effect correction
  and its regression-based evaluation are not applicable
- The goal is comparing peak-detection/quantitation accuracy across tools (e.g., coeluted
  or noisy peak handling) rather than assessing batch-effect correction
- Abundance drift must be assessed against something other than injection order (e.g.,
  RT drift), since this skill specifically fits BQC abundance vs injection order
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: MRMQuant
  role: Runs peak detection, quantitation, and QC-RLSC batch-effect correction on
    BQC samples; its GUI displays the BQC regression used for before/after evaluation
  repo: https://github.com/kslynn128171/MRMQuant
- name: MRMkit
  role: Source of the benchmark dataset (3793 samples, 33 batches, 295 compounds,
    389 BQCs) used to demonstrate correction effects
  repo: https://github.com/MRMkit/MRMkit
- name: LOESS
  role: Curve-fitting method underlying the batch-effect correction, fitted to compound
    abundance in QC samples
- name: MATLAB
  role: Runtime for MRMQuant; sample and method files are read from a binary MATLAB
    MAT-file
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/acs.analchem.4c02462
    title: 'MRMQuant: Automated MRM Data Quantitation for Large-Scale Targeted Metabolomics
      Analysis'
  source_content_hash: sha256:709f0f83828208ac96a58c9d2e2f3cf0672bf4efed499183fb9dc46bc22564f8
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/kslynn128171/MRMQuant
  repo_url: https://github.com/kslynn128171/MRMQuant
  tools:
  - LOESS
  - MATLAB
  - MRMQuant
  - MRMkit
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# bqc-regression-slope-r2-evaluation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Evaluates batch-effect correction quality in targeted MRM metabolomics by regressing batch-pooled QC (BQC) abundances against injection order and comparing regression slope and R2 before vs after QC-RLSC correction; successful correction flattens the BQC regression line.

## When to use

Use when a large multi-batch MRM dataset contains batch-pooled QC (BQC) samples and you need to quantify whether batch-effect correction (e.g., QC-RLSC) removed injection-order drift.

## When NOT to use

- The dataset contains no QC or batch-pooled QC samples, so batch-effect correction and its regression-based evaluation are not applicable
- The goal is comparing peak-detection/quantitation accuracy across tools (e.g., coeluted or noisy peak handling) rather than assessing batch-effect correction
- Abundance drift must be assessed against something other than injection order (e.g., RT drift), since this skill specifically fits BQC abundance vs injection order

## Inputs

- MRM sample files (mzML or vendor-converted) with 33 batches of samples including 389 batch-pooled QC (BQC) samples
- Method file with targeted compound information (295 compounds)
- Pre-correction compound abundances from peak detection and quantitation
- Post-correction abundances after QC-RLSC batch-effect correction

## Outputs

- Pre-correction BQC regression slope and R2 (e.g., slope 0.0004, R2 = 0.3)
- Post-correction BQC regression slope and R2 (e.g., slope ~0.0, R2 = 1.44e-4)
- batch_correction_results.csv compiling before/after slope and R2 per compound

## How to apply

Fit a linear regression of BQC sample abundances against injection order on the pre-correction abundances to record the baseline slope and R2 (in the MRMkit dataset, SM d18:1/C18:0 gave slope 0.0004 with R2 = 0.3). Apply the batch-effect correction (QC-RLSC in MRMQuant, excluding saturated peaks automatically from the regression because their abundances are unreliable), then refit the BQC-vs-injection-order regression on corrected abundances. Judge success by how close the post-correction slope is to 0.0 and how small the R2 is (expected ~0.0 and 1.44e-4 for SM d18:1/C18:0 in the MRMkit dataset of 3793 samples, 33 batches, 295 compounds, 389 BQCs). Compile before/after slope and R2 values into a results table for validation against reported values.

## Related tools

- **MRMQuant** (Runs peak detection, quantitation, and QC-RLSC batch-effect correction on BQC samples; its GUI displays the BQC regression used for before/after evaluation) — https://github.com/kslynn128171/MRMQuant
- **MRMkit** (Source of the benchmark dataset (3793 samples, 33 batches, 295 compounds, 389 BQCs) used to demonstrate correction effects) — https://github.com/MRMkit/MRMkit
- **LOESS** (Curve-fitting method underlying the batch-effect correction, fitted to compound abundance in QC samples)
- **MATLAB** (Runtime for MRMQuant; sample and method files are read from a binary MATLAB MAT-file)

## Evaluation signals

- Pre-correction BQC regression for SM d18:1/C18:0 reproduces slope 0.0004 with R2 = 0.3
- Post-correction BQC regression slope is ~0.0 with R2 = 1.44e-4, i.e., a near-flat line
- Saturated peaks are excluded from the regression (they have unreliable abundances)
- Recorded before/after slope and R2 values match the paper's reported results in batch_correction_results.csv
- The corrected regression line for BQC samples (red dots) against injection order shows no visible upward/downward trend

## Limitations

- Processing time for batch-effect correction of the MRMkit-scale dataset is around 5 h in MRMQuant (vs ~4 h for MRMKit), though actual computation time should be comparable
- Saturated peaks must be excluded from the regression because their abundances are unreliable
- The reported benchmark values (slope 0.0004, R2 = 0.3 before; ~0.0, 1.44e-4 after) are specific to SM d18:1/C18:0 in the MRMkit dataset; other compounds will have different baselines

## Evidence

- [results] BQC regression slope before/after correction: "the slope of the regression line of the BQC samples (red dots) was 0.0004 with R2 = 0.3 before the correction but was very close to a flat line (slope ∼0.0) with R2 = 1.44 × 10–4 after"
- [results] Saturated peak exclusion: "Saturated peaks are automatically excluded from the regression process for their unreliable abundances."
- [results] MRMkit dataset description: "the data set consists of 3793 samples in 33 batches, each with 295 compounds. A total of 389 batch-pooled QCs (BQCs)"
- [abstract] LOESS fit to QC abundances: "a LOESS curve is fitted to its abundance in the QC samples"
- [readme] Dataset repository: "We adopted the data set issued by MRMkit to demonstrate the effects before and after correction (https://github.com/MRMkit/MRMkit)."
