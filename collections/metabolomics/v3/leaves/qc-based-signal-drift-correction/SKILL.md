---
name: qc-based-signal-drift-correction
description: Use when a large-scale metabolomics or proteomics intensity matrix contains
  pooled QC samples injected repeatedly across the run and cumulative RSD plots show
  signal drift or batch effects that must be removed at the feature level before statistical
  analysis.
when_to_use_negative:
- Dataset contains no pooled QC injections across the run — use statTarget's shiftCor_dQC
  with Combat instead.
- Data have not yet been peak-picked (still raw instrument files); conversion and
  peak extraction must happen first via transX/XCMS/Skyline.
- Only single-batch data with no observable inter-batch variation — the batch-integration
  output of shiftCor is unnecessary overhead.
edam_operation: http://edamontology.org/operation_2944
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0092
tools:
- name: statTarget
  role: R package hosting the shiftCor function and QC-RFSC algorithm used for feature-level
    signal drift correction
  repo: https://github.com/13479776/statTarget
- name: shiftCor
  role: entry point applying the 80% rule, KNN imputation and QC-RFSC correction
  repo: https://github.com/13479776/statTarget
- name: transX
  role: converts XCMS/MZmine2/SIEVE/Skyline peak lists into statTarget meta and profile
    input formats
  repo: https://github.com/13479776/statTarget
- name: XCMS
  role: upstream peak-intensity extraction producing the profile file fed into shiftCor
- name: Skyline
  role: upstream targeted-peak extraction producing the profile file fed into shiftCor
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.aca.2018.08.002
    title: 'statTarget: A streamlined tool for signal drift correction and interpretations
      of quantitative mass spectrometry-based omics data'
  source_content_hash: sha256:a3ec4da66addb51923167f29b9435dd9411f9c8c4f5744b193375c37a35ef043
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/13479776/statTarget
  tools:
  - Skyline
  - XCMS
  - shiftCor
  - statTarget
  - transX
  edam_operation: http://edamontology.org/operation_2944
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# qc-based-signal-drift-correction

## Summary

Removes inter- and intra-batch signal drift at the feature level in LC-MS/GC-MS metabolomics and label-free proteomics by fitting a QC-based random forest signal correction (QC-RFSC) to pooled QC injections and dividing each feature by its per-sample correction factor, as implemented in the statTarget shiftCor function.

## When to use

Use when a large-scale metabolomics or proteomics intensity matrix contains pooled QC samples injected repeatedly across the run and cumulative RSD plots show signal drift or batch effects that must be removed at the feature level before statistical analysis.

## When NOT to use

- Dataset contains no pooled QC injections across the run — use statTarget's shiftCor_dQC with Combat instead.
- Data have not yet been peak-picked (still raw instrument files); conversion and peak extraction must happen first via transX/XCMS/Skyline.
- Only single-batch data with no observable inter-batch variation — the batch-integration output of shiftCor is unnecessary overhead.

## Inputs

- meta file with sample name, class, batch and injection order (statTarget samPeno format)
- peak intensity profile table exported from XCMS, MZmine2, SIEVE or Skyline (samFile)
- pooled QC sample labels denoted by the 'QC' name tag in the profile table

## Outputs

- QC-RFSC-corrected feature intensity matrix for all samples
- corrected matrices restricted to QC samples and to real samples only
- RSD/CV distribution of raw versus corrected peaks (per batch and overall)
- per-feature loplot quality-control images and shift statistics

## How to apply

Prepare a meta file (sample name, class, batch, order) and a profile file (peak intensities) exported from XCMS, MZmine2, SIEVE or Skyline; convert these with statTarget's transX function if needed. Label QC samples by name (tagged 'QC') and run shiftCor with Frule = 0.8, MLmethod = 'QCRFSC' and imputeM = 'KNN': the 80% rule keeps features with non-zero values in at least 80% of samples in any group, missing values are KNN-imputed, and an ensemble of ntree = 500 random forest regression trees grown on random sub-QC samples produces a per-feature, per-injection-order correction factor Fij. Divide each intensity by its factor (x'ij = xij/Fij), integrate multi-batch data, and export corrected matrices for all samples, QCs only and samples only together with RSD/CV distributions and per-feature loplot QC images. Validate by checking that the RSD distribution of QC peaks improves markedly — e.g. the proportion of QC peaks with RSD < 30% rose from about 43.9% (raw) to about 86.2% (corrected) for dataset 1.

## Related tools

- **statTarget** (R package hosting the shiftCor function and QC-RFSC algorithm used for feature-level signal drift correction) — https://github.com/13479776/statTarget
- **shiftCor** (entry point applying the 80% rule, KNN imputation and QC-RFSC correction) — https://github.com/13479776/statTarget
- **transX** (converts XCMS/MZmine2/SIEVE/Skyline peak lists into statTarget meta and profile input formats) — https://github.com/13479776/statTarget
- **XCMS** (upstream peak-intensity extraction producing the profile file fed into shiftCor)
- **Skyline** (upstream targeted-peak extraction producing the profile file fed into shiftCor)

## Examples

```
shiftCor(samPeno, samFile, Frule = 0.8, MLmethod = "QCRFSC", QCspan = 0, imputeM = "KNN")
```

## Evaluation signals

- Proportion of QC peaks with RSD < 30% must increase substantially after correction (e.g. 43.9% → 86.2% on dataset 1); flag failure if it does not.
- Number of peaks within 15% RSD should rise markedly — a 12.7-fold increase (to 25.1% of total peaks) was reported for dataset 1.
- RSD/CV distribution comparison raw vs corrected must be generated for each batch and overall; missing loplot images indicate incomplete run.
- Corrected output must be present as three matrices (all samples, QCs only, samples only) matching the original feature count.
- Verify the 80% rule was applied (Frule = 0.8) — features retained must have non-zero values in at least 80% of samples in any one group.

## Limitations

- Requires QC injections at regular intervals (every 5, 6 or 8 samples in the cited datasets); cannot correct drift without pooled QC samples.
- Performance depends on RF ensemble size (ntree = 500) and QCspan settings; the reported R-square is at median and first quartile, not exhaustive.
- Comparison methods QC-RLSC and QC-SVR also improve RSD but less than QC-RFSC, indicating the choice of MLmethod matters.
- Requires R ≥ 3.3.0; on macOS, XQuartz and RGtk2 (recommended 2.20.31 with R 3.3.0) must be installed for the GUI.

## Evidence

- [methods] RF ensemble size: [evidence span withheld — anchor status: not_found]
- [results] RSD improvement benchmark (dataset 1): [evidence span withheld — anchor status: not_found]
- [readme] statTarget repo URL: [evidence span withheld — anchor status: not_found]
