---
name: cubic-spline-injection-order-interpolation
description: Use when a large targeted-metabolomics run is split into batches with
  QC samples injected in acquisition order and you need to remove batch-effect drift
  from MRM quantitation, using the QC-RLSC approach of MRMQuant.
when_to_use_negative:
- When the dataset contains no QC samples injected across batches - MRMQuant conducts
  batch-effect correction only if quality control (QC) samples are included in the
  sample files.
- When the input is a raw MRM signal (mzML/abf) that has not yet been peak-detected
  and quantitated - this skill operates on the quantitated abundances, not on raw
  traces.
- When you need deconvolution of coeluted isomer peaks or peak-tip detection - that
  is a separate MRMQuant step, not the batch-effect correction itself.
edam_operation: http://edamontology.org/operation_3766
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0219
tools:
- name: MRMQuant
  role: Tool that implements the QC-RLSC batch-effect correction, fits the LOESS curve
    to QC samples and interpolates with cubic splines to remove response drift.
  repo: https://github.com/kslynn128171/MRMQuant
- name: MATLAB
  role: Runtime environment in which MRMQuant is implemented; also the format (binary
    MATLAB MAT-file) of the intermediate quantitation results loaded into the correction
    step.
- name: Microsoft Excel
  role: Output format (xlsx) in which the batch-effect-corrected quantitation results
    are written.
- name: ProteoWizard msconvert
  role: Upstream converter used to turn vendor raw MRM data into the mzML or MAT-file
    inputs that MRMQuant loads before quantitation and correction.
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1021/acs.analchem.4c02462
    title: 'MRMQuant: Automated MRM Data Quantitation for Large-Scale Targeted Metabolomics
      Analysis'
  source_content_hash: sha256:a8c66654065da84de091bc630e0d6b5cf8ff00cf62365ed584c5dc8874e333a1
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
  - MATLAB
  - MRMQuant
  - Microsoft Excel
  - ProteoWizard msconvert
  edam_operation: http://edamontology.org/operation_3766
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0219
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# cubic-spline-injection-order-interpolation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

MRMQuant's QC-RLSC batch-effect correction uses a cubic-spline interpolation/extrapolation of a LOESS curve fitted to QC-sample abundances (ordered by acquisition/injection order) to remove systematic response drift across sample batches. The interpolated trend is used to compute correction ratios that restore the abundances to the reference scale.

## When to use

Use when a large targeted-metabolomics run is split into batches with QC samples injected in acquisition order and you need to remove batch-effect drift from MRM quantitation, using the QC-RLSC approach of MRMQuant.

## When NOT to use

- When the dataset contains no QC samples injected across batches - MRMQuant conducts batch-effect correction only if quality control (QC) samples are included in the sample files.
- When the input is a raw MRM signal (mzML/abf) that has not yet been peak-detected and quantitated - this skill operates on the quantitated abundances, not on raw traces.
- When you need deconvolution of coeluted isomer peaks or peak-tip detection - that is a separate MRMQuant step, not the batch-effect correction itself.

## Inputs

- MRMQuant quantitation results (xlsx from Microsoft Excel or binary MATLAB MAT-file) containing abundances per compound per sample
- Method file with targeted compound information (transitions, RTs)
- Sample files annotated with which injections are QC / batch-pooled QC (BQC) samples
- MRM data converted from vendor raw files with ProteoWizard msconvert (mzML or MATLAB MAT-file)

## Outputs

- Corrected abundance matrix for all compounds and samples, saved as an xlsx file
- Regression diagnostics of BQC abundances vs injection order (slope and R2) before/after correction

## How to apply

Load the MRMQuant-quantitated abundances (MAT-file or xlsx quantitation output) and identify the QC samples within each batch (e.g. the batch-pooled QCs, BQCs, in the MRMkit 33-batch run). For each targeted compound, fit a LOESS curve to its abundance in the QC samples ordered by acquisition sequence. Use cubic-spline interpolation/extrapolation of that LOESS curve at every non-QC sample's injection point, so the fitted trend extends over the whole batch. Divide each sample's raw abundance by the interpolated QC trend value at its injection point to obtain a correction ratio; multiply the ratio by the reference sample's abundance so the corrected values stay on the original magnitude scale. The corrected matrix is batch_effect_corrected abundances (xlsx) ready for further analysis.

## Related tools

- **MRMQuant** (Tool that implements the QC-RLSC batch-effect correction, fits the LOESS curve to QC samples and interpolates with cubic splines to remove response drift.) — https://github.com/kslynn128171/MRMQuant
- **MATLAB** (Runtime environment in which MRMQuant is implemented; also the format (binary MATLAB MAT-file) of the intermediate quantitation results loaded into the correction step.)
- **Microsoft Excel** (Output format (xlsx) in which the batch-effect-corrected quantitation results are written.)
- **ProteoWizard msconvert** (Upstream converter used to turn vendor raw MRM data into the mzML or MAT-file inputs that MRMQuant loads before quantitation and correction.)

## Examples

```
MRMQuant.bat --input aging_bile_acids_MAT.mat --output batch_corrected_abundances.xlsx --batch-effect-correction QC-RLSC
```

## Evaluation signals

- After correction, the regression of batch-pooled QC (BQC) abundances against injection order should flatten: MRMQuant reports a change from slope 0.0004 with R2 = 0.3 before correction to slope ~0.0 with R2 = 1.44 × 10–4 after.
- The corrected abundances must remain on the original magnitude scale (restored via ratios times the reference sample's abundance), i.e. no global rescaling drift between raw and corrected outputs.
- Every non-QC sample should receive a finite corrected value: cubic-spline interpolation/extrapolation of the QC-LOESS curve with samples in acquiring order must cover all injection points, including samples before the first and after the last QC.
- Corrected results are written as a Microsoft Excel xlsx quantitation file (and/or the binary MATLAB MAT-file), with one corrected abundance per targeted compound per sample.
- Run-time sanity check: the MRMkit 3793-sample / 33-batch benchmark took ~5 h in MRMQuant vs ~4 h in MRMkit; a comparable-scale run should complete on that order of magnitude.

## Limitations

- The correction requires QC samples (ideally batch-pooled QCs) included in the sample files; without them MRMQuant does not run batch-effect correction.
- If a compound's QC abundance is saturated, the abundances are unreliable and should be excluded (saturated peaks are automatically excluded from regression in MRMQuant).
- The correction assumes QC-LOESS trend is a good proxy for drift in non-QC samples; it will not fix coelution-, baseline-, or deconvolution-related quantitation errors (TargetLynx, OpenMS, MRMkit and Skyline all had problematic quantitation of coeluted/noisy peaks in the same benchmark).
- Processing time scales with batch count and sample size (~5 h for the 3793-sample / 33-batch MRMkit data set, versus ~4 h for MRMkit).

## Evidence

- [other] Source card for the four-step QC-RLSC pipeline: [evidence span withheld — anchor status: not_found]
