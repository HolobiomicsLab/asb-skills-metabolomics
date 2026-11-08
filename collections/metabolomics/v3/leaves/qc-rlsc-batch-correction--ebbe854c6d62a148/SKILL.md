---
name: qc-rlsc-batch-correction
description: Use when MRM sample files contain QC or batch-pooled QC (BQC) samples
  and abundance plots show signal drifting with acquisition order across batches,
  requiring trend removal before downstream statistics.
when_to_use_negative:
- The sample set contains no QC or batch-pooled QC samples to anchor the LOESS trend,
  so there is no basis for robust local regression
- The goal is raw peak detection and quantitation rather than post-quantitation drift
  correction — run MRMQuant peak detection/quantitation first, then apply this skill
  to its abundances
- Abundance already shows a flat QC regression (slope ~0.0) with negligible R2, indicating
  no batch-effect drift to correct
edam_operation: http://edamontology.org/operation_3436
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0219
tools:
- name: MRMQuant
  role: 'Performs the QC-RLSC batch-effect correction: fits the LOESS curve to QC
    abundances, spline-interpolates the trend to non-QC samples, and restores corrected
    abundances'
  repo: https://github.com/kslynn128171/MRMQuant
- name: LOESS
  role: Locally weighted regression used to fit the response-drift trend of each compound
    across QC-sample injection order
- name: MATLAB
  role: Runtime for MRMQuant, including the built-in findpeaks function used in the
    upstream quantitation whose abundances feed the correction
  repo: https://github.com/kslynn128171/MRMQuant
- name: Microsoft Excel
  role: Format (xlsx) in which the corrected abundance results are saved for inspection
    and downstream use
- name: MRMkit
  role: Benchmark tool whose data set (3793 samples, 33 batches, 295 compounds, 389
    BQCs) was used to validate the correction against a flat BQC regression outcome
  repo: https://github.com/MRMkit/MRMkit
provenance:
  source_task_ids:
  - task_008
  - task_002
  source_papers:
  - doi: 10.1021/acs.analchem.4c02462
    title: 'MRMQuant: Automated MRM Data Quantitation for Large-Scale Targeted Metabolomics
      Analysis'
  source_content_hash: sha256:012d645da25091b679e07baf110dd1f244bde77d77a05309b6491bc064b50920
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
  - Microsoft Excel
  edam_operation: http://edamontology.org/operation_3436
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0219
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# qc-rlsc-batch-correction

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

QC-RLSC batch-effect correction in MRMQuant removes systematic injection-order drift in targeted MRM metabolomics by fitting a LOESS curve to QC-sample abundances per compound, interpolating the trend to all samples via cubic spline, and restoring abundances to the original magnitude scale.

## When to use

Use when MRM sample files contain QC or batch-pooled QC (BQC) samples and abundance plots show signal drifting with acquisition order across batches, requiring trend removal before downstream statistics.

## When NOT to use

- The sample set contains no QC or batch-pooled QC samples to anchor the LOESS trend, so there is no basis for robust local regression
- The goal is raw peak detection and quantitation rather than post-quantitation drift correction — run MRMQuant peak detection/quantitation first, then apply this skill to its abundances
- Abundance already shows a flat QC regression (slope ~0.0) with negligible R2, indicating no batch-effect drift to correct

## Inputs

- MRMQuant-quantitated abundance matrix for all compounds and samples (from the MATLAB MAT-file or xlsx quantitation output)
- Batch/injection-order metadata identifying which samples are QC or batch-pooled QC (BQC) runs
- MRM sample files and method file with targeted compound information as loaded into MRMQuant

## Outputs

- Batch-corrected abundance matrix for all compounds and samples (e.g. batch_corrected_abundances.xlsx)
- Regression plot of QC/BQC abundances versus injection order before and after correction for visual validation

## How to apply

Load the MRMQuant-quantitated abundances and identify QC (or BQC) samples within each of the acquisition batches. For each compound, fit a LOESS curve to its abundance in the QC samples across injection order, excluding saturated peaks automatically because their abundances are unreliable. Interpolate the fitted trend to every non-QC sample with cubic-spline interpolation in acquiring order, then compute ratios of each sample's chromatogram abundance to the LOESS-derived trend value. Restore corrected abundances to the original scale by multiplying the ratios by the reference sample's abundance, so corrected values remain comparable to raw data; skip regressed points whose ion chromatograms do not fit the curve by unchecking their box in the regression plot. Save the corrected abundance matrix for all compounds and samples (e.g. as an xlsx file) and verify the correction by regressing BQC abundances against injection order.

## Related tools

- **MRMQuant** (Performs the QC-RLSC batch-effect correction: fits the LOESS curve to QC abundances, spline-interpolates the trend to non-QC samples, and restores corrected abundances) — https://github.com/kslynn128171/MRMQuant
- **LOESS** (Locally weighted regression used to fit the response-drift trend of each compound across QC-sample injection order)
- **MATLAB** (Runtime for MRMQuant, including the built-in findpeaks function used in the upstream quantitation whose abundances feed the correction) — https://github.com/kslynn128171/MRMQuant
- **Microsoft Excel** (Format (xlsx) in which the corrected abundance results are saved for inspection and downstream use)
- **MRMkit** (Benchmark tool whose data set (3793 samples, 33 batches, 295 compounds, 389 BQCs) was used to validate the correction against a flat BQC regression outcome) — https://github.com/MRMkit/MRMkit

## Examples

```
Run MRMQuant in MATLAB with QC samples included in the loaded MRM sample files and method file, then export the corrected abundances to the xlsx output after batch-effect correction.
```

## Evaluation signals

- After correction, the regression of BQC abundances against injection order should flatten: expected change from slope 0.0004 with R2 = 0.3 (before) to slope ~0.0 with R2 = 1.44 × 10−4 (after) for SM d18:1/C18:0 in the MRMkit data set
- Corrected abundances must be restored to the original magnitude scale (ratios multiplied by the reference sample's abundance), so values remain directly comparable to raw chromatogram-derived abundances
- Saturated peaks must be excluded from the regression — confirm no saturated-peak abundances appear in the fitted QC trend
- The corrected matrix must cover every compound and sample in the original quantitation output with no dropped rows or columns
- The interpolated trend values at each sample's injection point must be positive, since they are denominators in the abundance-ratio computation

## Limitations

- Processing time for the batch-effect correction was around 5 h in the article's benchmark, longer than MRMkit's 4 h (though actual computation time is comparable); plan runtime accordingly for large batches
- Saturated peaks have unreliable abundances and are automatically excluded from the regression, which reduces the data available for trend fitting
- The user must visually judge the regression plot and may skip points whose ion chromatograms do not fit the curve — manual inspection is part of the workflow
- The correction requires QC samples in the sample files; without them MRMQuant does not perform batch-effect correction

## Evidence

- [results] Spline interpolation of the trend: [evidence span withheld — anchor status: not_found]
- [results] Abundance restoration step: [evidence span withheld — anchor status: not_found]
