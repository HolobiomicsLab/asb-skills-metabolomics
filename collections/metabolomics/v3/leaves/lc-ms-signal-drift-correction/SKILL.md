---
name: lc-ms-signal-drift-correction
description: Use when a preprocessed LC-MS peak table (e.g. MS-DIAL Excel export)
  contains pooled QC samples measured across the run and features show systematic
  intensity drift with injection order that must be corrected before statistical analysis.
when_to_use_negative:
- The dominant artifact is between-batch variation rather than within-run drift —
  use notame's batch effect correction (normalize_batches, dobc, ruvs_qc) instead
- You only hold raw LC-MS instrument files; notame operates on preprocessed peak tables
  produced by peak-picking software such as MS-DIAL
- A feature has fewer than 4 QC samples across the run, which the spline fit cannot
  support (it is flagged Missing_QCs, not corrected)
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: notame
  role: Provides the correct_drift() function and MetaboSet object model for the drift-correction
    workflow
  repo: https://github.com/hanhineva-lab/notame
- name: smooth.spline
  role: R function used to fit the cubic spline model of log-abundance versus injection
    order for QC samples
- name: R
  role: Runtime environment for notame (requires R >= 3.5.0) in which the spline fitting
    and correction are executed
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.3390/metabo10040135
    title: '“Notame”: Workflow for Non-Targeted LC–MS Metabolic Profiling'
  source_content_hash: sha256:9e70807125606c756e6c104930fd5de05cd85b0537221986d8fd960acbeef568
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/hanhineva-lab/notame
  tools:
  - R
  - notame
  - smooth.spline
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# lc-ms-signal-drift-correction

## Summary

Corrects systematic signal intensity drift in untargeted LC-MS feature tables using notame's cubic-spline correction, which models log-transformed QC abundances as a function of injection order and additively adjusts every sample before back-transformation.

## When to use

Use when a preprocessed LC-MS peak table (e.g. MS-DIAL Excel export) contains pooled QC samples measured across the run and features show systematic intensity drift with injection order that must be corrected before statistical analysis.

## When NOT to use

- The dominant artifact is between-batch variation rather than within-run drift — use notame's batch effect correction (normalize_batches, dobc, ruvs_qc) instead
- You only hold raw LC-MS instrument files; notame operates on preprocessed peak tables produced by peak-picking software such as MS-DIAL
- A feature has fewer than 4 QC samples across the run, which the spline fit cannot support (it is flagged Missing_QCs, not corrected)

## Inputs

- Preprocessed LC-MS peak table from MS-DIAL (Excel spreadsheet read via notame::read_from_excel)
- MetaboSet object with sample metadata including injection order, group column (e.g. Group), and QC-sample annotations
- Feature abundance matrix with zeros marked as missing (mark_nas value = 0)

## Outputs

- Drift-corrected abundance matrix (e.g. drift_corrected_exprs.csv with one row per sample and one column per feature)
- Per-feature DC_note annotation column with values Drift_corrected, Low-quality, Missing_QCs, or Negative_DC
- Updated MetaboSet object with corrected exprs slot

## How to apply

Read the peak table with notame::read_from_excel (e.g. the bundled sample_data_whole.xlsx with corner_row = 4, corner_column = "X", split_by = c("Column", "Ion mode")) and build MetaboSet objects with construct_metabosets, then convert zero abundances to NA with mark_nas(value = 0). Log-transform the exprs matrix because the spline model is fit in log space to satisfy its assumptions. For each feature, fit R's smooth.spline to the QC samples' log-abundance versus injection order, selecting the smoothing parameter by leave-one-out cross validation in the recommended interval [0.5, 1.5]; features with fewer than 4 QC samples are recorded as Missing_QCs instead of corrected. Apply the additive correction x_corrected(i) = x_original(i) + mean_QC − f_drift(i) in log space to every sample, back-transform to linear scale, and annotate each feature with a DC_note of Drift_corrected, Low-quality, Missing_QCs, or Negative_DC depending on whether QC quality improved, QCs were insufficient, or negative values arose. The method follows Kirwan & Broadhurst et al., cited in the notame README as the basis for its cubic-spline drift correction.

## Related tools

- **notame** (Provides the correct_drift() function and MetaboSet object model for the drift-correction workflow) — https://github.com/hanhineva-lab/notame
- **smooth.spline** (R function used to fit the cubic spline model of log-abundance versus injection order for QC samples)
- **R** (Runtime environment for notame (requires R >= 3.5.0) in which the spline fitting and correction are executed)

## Examples

```
set_log <- notame::mark_nas(set, value = 0); corrected <- notame::correct_drift(set_log)
```

## Evaluation signals

- Output file drift_corrected_exprs.csv exists with exactly one row per sample and one column per feature
- A DC_note column is present and its values are limited to Drift_corrected, Low-quality, Missing_QCs, and Negative_DC
- For features annotated Drift_corrected, the spread of QC-sample values across injection order is reduced relative to the original matrix
- Every feature with fewer than 4 QC samples carries the Missing_QCs note rather than a spline correction
- Back-transformed corrected abundances are non-negative; any negative values are flagged with the Negative_DC note

## Limitations

- Spline fitting requires at least 4 QC samples per feature; features lacking sufficient QCs are recorded as Missing_QCs and left uncorrected
- The additive log-space correction can produce negative values after back-transformation, which are flagged as Negative_DC
- Correction may not improve feature quality, in which case the feature is flagged Low-quality rather than Drift_corrected
- The notame API is described as experimental with possible breaking changes; the original antonvsdata/notame repository is no longer maintained (moved to hanhineva-lab/notame)

## Evidence

- [methods] Untargeted LC-MS metabolomics data usually suffers from signal intensity drift: "Untargeted LC-MS metabolomics data usually suffers from signal intensity drift"
- [methods] correct_drift function: "To run drift correction on your data, use the function ```correct_drift```."
- [methods] smooth.spline modelling: "The R function \code{smooth.spline} performs the modelling"
- [readme] cubic spline drift correction in README: [evidence span withheld — anchor status: external_doc]
- [readme] notame input scope: "This package can be used to analyze preprocessed LC-MS data in non-targeted metabolomics."
