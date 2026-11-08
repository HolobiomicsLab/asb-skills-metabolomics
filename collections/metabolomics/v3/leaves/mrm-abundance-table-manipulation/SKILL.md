---
name: mrm-abundance-table-manipulation
description: Use when a targeted MRM dataset contains QC samples per batch and you
  need to reduce systematic injection-order drift in compound abundances, e.g. before
  downstream statistical or pathway analysis.
when_to_use_negative:
- The dataset contains no QC samples in the sample files (MRMQuant skips batch-effect
  correction when no QCs are present).
- The input is raw vendor MRM data rather than a quantitated abundance matrix — first
  run MRMQuant quantitation to produce abundances before applying this correction.
- The analysis goal is deconvolution of coeluted peaks or absolute quantitation via
  standard curves, which are separate MRMQuant features rather than abundance-table
  drift correction.
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: MRMQuant
  role: Produces the quantitated abundance matrix and includes batch-effect correction
    based on QC samples; the LOESS/spline correction is applied to its xlsx/MAT output.
  repo: https://github.com/kslynn128171/MRMQuant
- name: LOESS
  role: Fits the response drift curve to the compound's abundance in QC samples across
    injection order.
- name: MATLAB
  role: Runtime environment; MRMQuant generates a binary MATLAB MAT-file that can
    be loaded into the correction workflow.
- name: Microsoft Excel
  role: Receives and stores the corrected abundance matrix as batch_corrected_abundances.xlsx.
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1021/acs.analchem.4c02462
    title: 'MRMQuant: Automated MRM Data Quantitation for Large-Scale Targeted Metabolomics
      Analysis'
  source_content_hash: sha256:e9a1af416326a04aa6333f85ae6bf00bc9bcc5b84330844850e24a63192b00d9
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
  - Microsoft Excel
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# mrm-abundance-table-manipulation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Manipulate the MRMQuant-quantitated abundance matrix (from the MAT-file or xlsx output) to fit LOESS curves on QC samples, interpolate QC trends via cubic splines across injection order, and rescale non-QC abundances to restore the reference sample's magnitude.

## When to use

Use when a targeted MRM dataset contains QC samples per batch and you need to reduce systematic injection-order drift in compound abundances, e.g. before downstream statistical or pathway analysis.

## When NOT to use

- The dataset contains no QC samples in the sample files (MRMQuant skips batch-effect correction when no QCs are present).
- The input is raw vendor MRM data rather than a quantitated abundance matrix — first run MRMQuant quantitation to produce abundances before applying this correction.
- The analysis goal is deconvolution of coeluted peaks or absolute quantitation via standard curves, which are separate MRMQuant features rather than abundance-table drift correction.

## Inputs

- MRMQuant quantitated abundance matrix for all compounds and samples (from the MAT-file or xlsx quantitation output)
- Batch/QC sample annotation per injection order

## Outputs

- batch_corrected_abundances.xlsx — the drift-corrected abundance matrix for all compounds and samples

## How to apply

Load the quantitated abundances for all compounds and samples (MRMQuant saves them in an xlsx file after generating a binary MATLAB MAT-file), then identify the QC samples within each batch. For each compound, fit a LOESS curve to its abundance in the QC samples across injection order to capture the response drift trend. Use cubic-spline interpolation to carry the fitted QC trend to every non-QC sample's injection point, and divide the sample's raw abundance by the interpolated trend value to compute the correction ratio. Restore the corrected abundances by multiplying the ratios by the reference sample's abundance so values remain directly comparable to the raw data; save the resulting matrix as batch_corrected_abundances.xlsx for subsequent inspection and analysis.

## Related tools

- **MRMQuant** (Produces the quantitated abundance matrix and includes batch-effect correction based on QC samples; the LOESS/spline correction is applied to its xlsx/MAT output.) — https://github.com/kslynn128171/MRMQuant
- **LOESS** (Fits the response drift curve to the compound's abundance in QC samples across injection order.)
- **MATLAB** (Runtime environment; MRMQuant generates a binary MATLAB MAT-file that can be loaded into the correction workflow.)
- **Microsoft Excel** (Receives and stores the corrected abundance matrix as batch_corrected_abundances.xlsx.)

## Examples

```
mrquant --input abundances.xlsx --qc-file qc_annotation.csv --output batch_corrected_abundances.xlsx
```

## Evaluation signals

- After correction, the regression slope of batch-pooled QC (BQC) abundances against injection order should flatten to ~0.0, and R^2 should drop to the order of 10^-4 (reported: slope 0.0004 → ~0.0, R2 0.3 → 1.44 × 10^-4).
- Corrected abundances must be restored to the original magnitude scale, so sample abundances should remain directly comparable to the pre-correction raw values rather than being shifted to an arbitrary scale.
- All compounds and all samples present in the input abundance matrix must appear in the output batch_corrected_abundances.xlsx.
- For every non-QC sample, the corrected value equals raw abundance divided by the interpolated QC trend value at its injection position, multiplied by the reference sample's abundance.

## Limitations

- Correction relies on QC samples being included in the sample files; without QCs, MRMQuant does not perform batch-effect correction.
- Processing time for the batch-effect correction is around 5 h on large MRM datasets (e.g. 3793 samples in 33 batches with 295 compounds), so planning is required for large cohorts.
- The correction is calibrated per compound and per batch; compounds whose QC abundance trend is poorly sampled may not benefit from the spline interpolation step.

## Evidence
