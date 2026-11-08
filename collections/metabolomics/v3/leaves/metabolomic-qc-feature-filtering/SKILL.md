---
name: metabolomic-qc-feature-filtering
description: Use when a metabonomic SummarizedExperiment (metabo_SE) and its per-feature
  CV table (metabo_CV) need filtering of non-reproducible metabolic features prior
  to downstream MWAS or STOCSY analysis.
when_to_use_negative:
- When the metabolomic matrix has not yet been paired with a per-feature CV table
  (metabo_CV) from an upstream CV-analysis step.
- When the goal is statistical association modelling (e.g. MWAS models, bootstrapping)
  rather than QC removal of non-reproducible features.
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_3572
tools:
- name: MWASTools
  role: Provides the CV_filter function used to apply the CV_th = 0.30 cut-off and
    return the filtered SummarizedExperiment.
  repo: https://bioconductor.org/packages/MWASTools/
- name: R
  role: Runtime environment in which MWASTools and the CV filtering workflow are executed
    (R version >= 3.4).
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btx477
    title: 'MWASTools: an R/bioconductor package for metabolome-wide association studies'
  source_content_hash: sha256:b239492a519a58036c019996e130c8c3c25cb5867d0dc5657503b23668e0369b
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MWASTools
  - R
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_3572
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# metabolomic-qc-feature-filtering

## Summary

Filters non-reproducible metabolic features from a metabonomic dataset by applying a coefficient-of-variation (CV) threshold (CV_th = 0.30) via the MWASTools CV_filter mechanism, retaining only features with CV <= 0.30 in the SummarizedExperiment object metabo_SE.

## When to use

Use when a metabonomic SummarizedExperiment (metabo_SE) and its per-feature CV table (metabo_CV) need filtering of non-reproducible metabolic features prior to downstream MWAS or STOCSY analysis.

## When NOT to use

- When the metabolomic matrix has not yet been paired with a per-feature CV table (metabo_CV) from an upstream CV-analysis step.
- When the goal is statistical association modelling (e.g. MWAS models, bootstrapping) rather than QC removal of non-reproducible features.

## Inputs

- SummarizedExperiment object (metabo_SE) holding the metabolomic feature matrix
- per-feature coefficient-of-variation table (metabo_CV) from the upstream CV-analysis step

## Outputs

- Filtered SummarizedExperiment object (reassigned to metabo_SE) containing only reproducible features
- RDS file of the filtered metabo_SE
- Count of retained features and their ppm values

## How to apply

Load the SummarizedExperiment object metabo_SE containing the metabolomic feature matrix along with the per-feature CV table metabo_CV produced by the upstream CV-analysis step. Call MWASTools::CV_filter(metabo_SE, metabo_CV, CV_th = 0.30) so that only features with coefficient of variation <= 0.30 are retained; the function returns a filtered SummarizedExperiment that is reassigned to metabo_SE. Save the resulting object as an RDS file and record the number of retained features and their ppm values. Verify correctness by checking that every retained feature's CV value in the CV table is <= 0.30 and that all removed features exceed the threshold, then feed the filtered matrix into downstream metabolome-wide association analysis.

## Related tools

- **MWASTools** (Provides the CV_filter function used to apply the CV_th = 0.30 cut-off and return the filtered SummarizedExperiment.) — https://bioconductor.org/packages/MWASTools/
- **R** (Runtime environment in which MWASTools and the CV filtering workflow are executed (R version >= 3.4).)

## Examples

```
metabo_SE <- MWASTools::CV_filter(metabo_SE, metabo_CV, CV_th = 0.30)
```

## Evaluation signals

- Every retained feature's CV value in the CV table is <= 0.30.
- All removed features have CV values exceeding the 0.30 threshold.
- The output object is a valid SummarizedExperiment reassigned to metabo_SE and can be written to an RDS file.
- Recorded retained-feature count and ppm values are consistent with the filtered matrix dimensions.

## Limitations

- The cut-off is hard-coded in this workflow as CV_th = 0.30; other thresholds require overriding the default argument.
- Filtering is purely reproducibility-based (CV-driven); it does not address missingness, batch effects, or signal-to-noise, which are separate QC concerns in MWASTools.
- Requires an already-computed per-feature CV table (metabo_CV); it cannot be applied to raw spectra alone.

## Evidence

- [supplementary] CV_filter mechanism applies a CV cut-off (CV_th = 0.30) to filter the metabolic matrix, returning a filtered SummarizedExperiment reassigned to metabo_SE.: [evidence span withheld — anchor status: not_found]
- [abstract] MWASTools provides QC analysis as part of its integrated pipeline.: "Key functionalities of our package include: quality control analysis; metabolome-wide association analysis using various models (partial correlations, generalized linear models)"
- [supplementary] Workflow uses SummarizedExperiment metabo_SE and CV table metabo_CV with CV_th = 0.30.: [evidence span withheld — anchor status: not_found]
