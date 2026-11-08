---
name: coefficient-of-variation-computation-across-datasets
description: Use when comparing peak-area reproducibility of an untargeted metabolomics
  peak list before vs. after a peak-curation or reintegration step, e.g. to verify
  that PeakDetective reintegration of artifact peaks lowers the CV of peak areas.
when_to_use_negative:
- Input is a single-group dataset with no replicate structure — CV across samples
  is meaningless without comparable within-group variability context
- Peak areas contain imputed zeros or missing values that were not excluded, since
  CV = SD/mean is distorted by imputation (e.g. imputeRowMin output) and by near-zero
  means
edam_operation: http://edamontology.org/operation_3435
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: PeakDetective
  role: Performs the artifact-aware peak reintegration whose effect on CV is being
    quantified; supplies reintegrated peak areas via findPeakBoundaries/integratePeak
  repo: https://github.com/pattilab/PeakDetective
- name: numpy
  role: Computes SD and mean per feature to derive CV of peak areas
- name: pandas
  role: Loads peak-area matrices and feature tables from CSV and organizes per-feature
    CV computation
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1021/acs.analchem.3c00764
    title: 'PeakDetective: A Semisupervised Deep Learning-Based Approach for Peak
      Curation in Untargeted Metabolomics'
  source_content_hash: sha256:14bc74637c3147c62950188bafede99f6183792220041522732c95c6e1db3543
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/pattilab/PeakDetective
  tools:
  - PeakDetective
  - Python
  - numpy
  - pandas
  - pyteomics
  edam_operation: http://edamontology.org/operation_3435
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# coefficient-of-variation-computation-across-datasets

## Summary

Compute per-feature coefficient of variation (CV = SD/mean) of LC/MS peak areas across Datasets 1-3 to quantify peak boundary quality before and after PeakDetective's artifact-aware peak reintegration, as reported in Figure S10 of the PeakDetective paper.

## When to use

Use when comparing peak-area reproducibility of an untargeted metabolomics peak list before vs. after a peak-curation or reintegration step, e.g. to verify that PeakDetective reintegration of artifact peaks lowers the CV of peak areas.

## When NOT to use

- Input is a single-group dataset with no replicate structure — CV across samples is meaningless without comparable within-group variability context
- Peak areas contain imputed zeros or missing values that were not excluded, since CV = SD/mean is distorted by imputation (e.g. imputeRowMin output) and by near-zero means

## Inputs

- Original XCMS peak-area matrix per dataset (features x samples, CSV)
- Reintegrated peak-area matrix from PeakDetective's artifact-aware reintegration (features x samples)
- Feature annotations (mz, rt) matching the XCMS peak list

## Outputs

- Per-feature CV values before and after reintegration for Datasets 1-3
- CV distribution comparison plot (figure_s10_cv_comparison.png) analogous to Figure S10
- CSV file of reintegrated peak areas

## How to apply

Load the peak-area matrix per dataset (features x samples) as a pandas DataFrame and compute CV = SD/mean for each feature row with numpy, treating samples as replicates for variability. Compute CVs twice: once on the original XCMS peak areas and once on the reintegrated areas produced by PeakDetective's findPeakBoundaries/integratePeak workflow (artifact peaks re-integrated using mean peak boundaries from non-artifact samples, defaultWidth = 0.5 min, EICs at 60 points/min over a 1.0 min window). Plot the before/after CV distributions for Datasets 1-3 and check that the post-reintegration distribution is shifted lower, reproducing Figure S10. Save the reintegrated peak areas to CSV and the comparison plot (e.g. figure_s10_cv_comparison.png).

## Related tools

- **PeakDetective** (Performs the artifact-aware peak reintegration whose effect on CV is being quantified; supplies reintegrated peak areas via findPeakBoundaries/integratePeak) — https://github.com/pattilab/PeakDetective
- **numpy** (Computes SD and mean per feature to derive CV of peak areas)
- **pandas** (Loads peak-area matrices and feature tables from CSV and organizes per-feature CV computation)

## Examples

```
cv_before = df_areas.std(axis=1) / df_areas.mean(axis=1); cv_after = df_reint.std(axis=1) / df_reint.mean(axis=1); df_reint.to_csv('reintegrated_peak_areas.csv')
```

## Evaluation signals

- Post-reintegration peak-area CV distribution is shifted lower than the pre-reintegration distribution in all three datasets (Datasets 1-3), matching the qualitative trend of Figure S10
- CV values are finite and non-negative for all retained features; features with near-zero mean peak areas are excluded or flagged
- The number of features in the before/after CV tables matches the curated XCMS peak list after scoring with the PeakDetective classifier at cutoff = 0.8
- Reintegrated peak areas are written to CSV with the same feature/sample schema as the input peak areas

## Limitations

- No numeric CV threshold is reported in the paper; validation of the reduction is a qualitative distribution shift check against Figure S10
- CV is computed per feature within a dataset; it does not by itself separate true biological variability from artifact-induced variability — the artifact/true peak labels come from the PeakDetective classifier
- Reintegration depends on well-defined mean peak boundaries from non-artifact samples; features with too few non-artifact samples may not benefit

## Evidence

- [supplementary] Artifact-aware reintegration improves boundary definition: [evidence span withheld — anchor status: not_found]
