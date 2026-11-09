---
name: loess-drift-trend-fitting
description: Use when MRM sample files contain QC samples and systematic signal drift
  across injection order must be corrected per compound, e.g. BQC regression slope
  != 0 before correction.
when_to_use_negative:
- The sample files contain no QC samples — MRMQuant only conducts batch-effect correction
  'if quality control (QC) samples are included in the sample files'
- The abundances are not yet quantitated per compound per sample (peak detection and
  quantitation must precede drift correction)
- The goal is aligning retention times or quantifying coeluted peaks rather than removing
  injection-order drift — use MRMQuant's deconvolution/quantitation steps instead
edam_operation: null
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: MRMQuant
  role: Host tool that performs LOESS fitting to QC abundances and QC-RLSC-style batch-effect
    correction on MRM data
  repo: https://github.com/kslynn128171/MRMQuant
- name: LOESS
  role: Locally weighted regression method fitted to the compound's abundance in QC
    samples to estimate response drift
- name: MATLAB
  role: Runtime environment in which MRMQuant and its built-in functions (e.g. findpeaks)
    execute; MAT-file is the intermediate format
- name: MRMkit
  role: Comparison tool whose benchmark data set (3793 samples, 33 batches, 295 compounds,
    389 BQCs) demonstrated the before/after correction effect
  repo: https://github.com/MRMkit/MRMkit
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1021/acs.analchem.4c02462
    title: 'MRMQuant: Automated MRM Data Quantitation for Large-Scale Targeted Metabolomics
      Analysis'
  source_content_hash: sha256:5cc3d91c6d3a2109f08daa636e85dc16e1a1628fa2c68a459c4d92ed70f42d40
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# loess-drift-trend-fitting

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Fits a LOESS curve to a compound's abundance across QC samples in acquisition order within MRMQuant, then interpolates/extrapolates that trend via cubic spline to estimate and remove instrument/batch drift from other samples.

## When to use

Use when MRM sample files contain QC samples and systematic signal drift across injection order must be corrected per compound, e.g. BQC regression slope != 0 before correction.

## When NOT to use

- The sample files contain no QC samples — MRMQuant only conducts batch-effect correction 'if quality control (QC) samples are included in the sample files'
- The abundances are not yet quantitated per compound per sample (peak detection and quantitation must precede drift correction)
- The goal is aligning retention times or quantifying coeluted peaks rather than removing injection-order drift — use MRMQuant's deconvolution/quantitation steps instead

## Inputs

- MRMQuant-quantitated abundance matrix for all compounds and samples (MAT-file or xlsx quantitation output)
- Identification of QC samples (e.g. batch-pooled QCs, BQCs) within each batch and their acquiring order
- MRM sample files with method file (MRMQuant input; e.g. mzML from ProteoWizard msconvert)

## Outputs

- Drift-corrected abundance matrix for all compounds and samples (e.g. saved as batch_corrected_abundances.xlsx)
- Fitted LOESS QC trend per compound per batch, with correction ratios applied to non-QC samples

## How to apply

Within MRMQuant's batch-effect correction, first fit a LOESS curve to the compound's abundance in the QC samples across their acquiring order. Then use cubic-spline interpolation to interpolate (or extrapolate) the fitted QC trend to every non-QC sample injection point in the batch. Divide each sample's raw abundance by the interpolated QC trend value to compute the correction ratio, and after trend removal restore abundances by multiplying the ratios by the reference sample's abundance. The rationale is that QC injections sample the same material repeatedly, so their LOESS-fitted abundance trace estimates the instrument response drift; removing it flattens BQC regression over injection order (from slope 0.0004 with R2 = 0.3 to slope ~0.0 with R2 = 1.44 × 10–4 in the MRMkit 33-batch data set). For ~3793 samples in 33 batches across 295 compounds, MRMQuant completes this correction in roughly 5 h.

## Related tools

- **MRMQuant** (Host tool that performs LOESS fitting to QC abundances and QC-RLSC-style batch-effect correction on MRM data) — https://github.com/kslynn128171/MRMQuant
- **LOESS** (Locally weighted regression method fitted to the compound's abundance in QC samples to estimate response drift)
- **MATLAB** (Runtime environment in which MRMQuant and its built-in functions (e.g. findpeaks) execute; MAT-file is the intermediate format)
- **MRMkit** (Comparison tool whose benchmark data set (3793 samples, 33 batches, 295 compounds, 389 BQCs) demonstrated the before/after correction effect) — https://github.com/MRMkit/MRMkit

## Evaluation signals

- After correction, regression of batch-pooled QC (BQC) abundances against injection order should flatten: slope should approach 0 and R2 drop drastically (MRMkit benchmark: slope 0.0004, R2 = 0.3 before → slope ~0.0, R2 = 1.44 × 10–4 after)
- Every non-QC sample must have received an interpolated/extrapolated QC trend value at its injection point — check that the correction ratio was computed for all samples, not only QCs
- Corrected abundances should remain on the original magnitude scale (ratios multiplied back by the reference sample's abundance), comparable to raw data
- Output abundance matrix should cover all compounds (e.g. 295 compounds in the benchmark) and all samples in the batch with no missing values after correction

## Limitations

- Requires QC samples included in the batch; correction cannot be performed without them
- Extrapolation of the LOESS/cubic-spline trend to sample injections outside the QC range relies on the fitted curve and may be less reliable at batch edges
- Processing time grows with batch size — around 5 h for the 3793-sample, 33-batch MRMkit data set (comparable in actual computation time to MRMKit's 4 h)

## Evidence
