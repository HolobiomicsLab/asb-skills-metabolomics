---
name: gam-based-drift-correction-on-qc-samples
description: Use when a MRMhubExperiment holds imported MRM intensity data whose QC-injected
  features show instrument drift over the run sequence, after normalize_by_istd()
  and before batch correction.
when_to_use_negative:
- Input is still raw instrument data (mzML/vendor files) that first needs consensus
  peak integration by the INTEGRATOR module — drift correction operates on an imported
  intensity table.
- The observed variation is between-batch rather than within-sequence drift — batch
  correction (correct_batch_combat or correct_batch_serrf) is the appropriate step
  instead.
- There are no annotated reference QC samples in the experiment, so no trend can be
  fitted on ref_qc_types.
tools:
- name: mrmhub
  role: R package (QUANT module) supplying correct_drift_gam() and the MRMhubExperiment
    data container for drift correction
  repo: https://github.com/SLINGhub/MRMhub
- name: mgcv
  role: Optional smoother backend fitting the GAM trend on QC samples (bs, k, sp parameters)
- name: MRMhub / INTEGRATOR
  role: Upstream producer of integrated peak areas that are imported via import_data_mrmhub()
    before drift correction
  repo: https://github.com/SLINGhub/MRMhub
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/acs.analchem.0c03060
    title: 'MRMkit: Automated Data Processing for Large-Scale Targeted Metabolomics
      Analysis'
  source_content_hash: sha256:4683eef4c0bb6144628f2e381f8667229383ea097574e8c14bc50349f313b582
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/SLINGhub/MRMhub
  tools:
  - MRMhub / INTEGRATOR
  - correct_drift_gam
  - get_analyticaldata
  - mgcv
  - mrmhub
  - ranger
  - sva
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6153846153846154
n_claims: 26
---

# GAM-based drift correction on QC samples

## Summary

Corrects intensity drift across an MRM analysis sequence by fitting a smoother (generalized additive model via the mgcv backend in mrmhub's correct_drift_gam) on reference QC samples and adjusting feature intensities batch-wise, as one step in the MRMhub QUANT post-processing pipeline between ISTD normalization and batch correction.

## When to use

Use when a MRMhubExperiment holds imported MRM intensity data whose QC-injected features show instrument drift over the run sequence, after normalize_by_istd() and before batch correction.

## When NOT to use

- Input is still raw instrument data (mzML/vendor files) that first needs consensus peak integration by the INTEGRATOR module — drift correction operates on an imported intensity table.
- The observed variation is between-batch rather than within-sequence drift — batch correction (correct_batch_combat or correct_batch_serrf) is the appropriate step instead.
- There are no annotated reference QC samples in the experiment, so no trend can be fitted on ref_qc_types.

## Inputs

- MRMhubExperiment object with imported peak-area intensities (e.g. from INTEGRATOR results, MRMhub TSV, MassHunter CSV, or Skyline long-format CSV reports)
- Internal-standard-normalized intensity variable (norm_intensity) produced by normalize_by_istd()
- Sample-type annotations identifying reference QC samples (ref_qc_types) and batch assignments

## Outputs

- MRMhubExperiment with a drift-corrected intensity variable per feature
- Fitted trend curves stored in the experiment for subsequent batch correction (correct_batch_combat / correct_batch_serrf)
- Processed quantitative matrix retrievable via get_analyticaldata() and exportable as processed_dataset.tsv

## How to apply

Operate on the MRMhubExperiment container produced by import_data_mrmhub(), first applying normalize_by_istd() so the drift variable is internal-standard-normalized. Call correct_drift_gam() on the target variable (e.g. 'norm_intensity'), fitting the trend on reference QC sample types (ref_qc_types) with mgcv as the optional smoother backend, tuning the smoother via bs, k, and sp and processing batch_wise so each defined batch gets its own trend. Use replace_previous to overwrite an existing correction rather than stacking trend curves, and ignore_istd to keep internal-standard features untouched. If the GAM fit is unsuitable, the same drift-correction slot accepts alternative smoothers such as correct_drift_cubicspline, correct_drift_loess, or correct_drift_gaussiankernel, all sharing the ref_qc_types/batch_wise/replace_previous interface. Downstream, feed the drift-corrected variable into correct_batch_combat() or correct_batch_serrf(), then retrieve the processed matrix via get_analyticaldata() and export to processed_dataset.tsv.

## Related tools

- **mrmhub** (R package (QUANT module) supplying correct_drift_gam() and the MRMhubExperiment data container for drift correction) — https://github.com/SLINGhub/MRMhub
- **mgcv** (Optional smoother backend fitting the GAM trend on QC samples (bs, k, sp parameters))
- **MRMhub / INTEGRATOR** (Upstream producer of integrated peak areas that are imported via import_data_mrmhub() before drift correction) — https://github.com/SLINGhub/MRMhub

## Examples

```
mexp <- mrmhub::normalize_by_istd(mexp) |> mrmhub::correct_drift_gam()
```

## Evaluation signals

- Confirm the drift-corrected variable exists per feature and that replace_previous semantics were honored (no stacked/duplicated trend curves when re-running).
- Check internal-standard features were not altered (ignore_istd) and that QC annotations (ref_qc_types) covered all reference QC samples used for fitting.
- Verify the correction was applied batch_wise — each defined batch should show its own fitted trend rather than one global curve.
- Compare QC replicate CVs before vs. after correction: drift-induced CV inflation within a batch should shrink on the corrected variable.
- Confirm the corrected variable flows into batch correction and is retrievable via get_analyticaldata(), exported as processed_dataset.tsv.

## Limitations

- The mrmhub package installs specialised functions' dependencies lazily — the mgcv backend may need to be installed on first use.
- Alternative drift smoothers (cubicspline, loess, gaussiankernel) expose different tuning parameters (spar/lambda, span/degree, kernel_size); a poor GAM choice of bs/k/sp can under- or over-fit the QC trend, and the shared interface offers conditional_correction and recalc_trend_after options that must be set deliberately.
- Drift correction presumes reference QC samples bracket the run sequence; population-scale claims rest on reproducible pipelines with a full digital footprint, so unrecorded manual parameter changes break traceability.

## Evidence

- [methods] Drift correction slot shares ref_qc_types/batch_wise interface: "correct_drift_cubicspline(data, variable, ref_qc_types, batch_wise, ignore_istd, replace_previous, cv, spar, lambda, penalty, conditional_correction, recalc_trend_after, log_transform_internal,"
- [readme] QUANT module role for tailored post-processing pipelines: [evidence span withheld — anchor status: external_doc]
- [readme] Experiment container flow through import and ISTD normalization: "mexp <- MRMhubExperiment() |>
  import_data_mrmhub(path = demo_file, import_metadata = TRUE) |>
  normalize_by_istd()"
- [readme] Correction and QC metric functions with feature filtering: [evidence span withheld — anchor status: external_doc]
