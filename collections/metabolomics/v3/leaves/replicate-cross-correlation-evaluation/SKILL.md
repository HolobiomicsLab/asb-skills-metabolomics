---
name: replicate-cross-correlation-evaluation
description: Use when you have a multi-batch metabolomics feature matrix with replicate
  reference samples (e.g. pooled or NIST 1950 SRM samples) present in each batch and
  need to quantify whether a normalization removed batch effects.
when_to_use_negative:
- The dataset has no replicate reference samples (e.g. pooled QC or NIST 1950 SRM)
  present across batches, so no cross-batch replicate pairs can be matched via the
  'group' column
- You need sample-level biological discrimination rather than batch-effect assessment
  — benchmark groups are excluded from regularization and only support model evaluation
- Evaluating spectral preservation or variance inflation, where batch-wise VC and
  fraction of features with VC inflated by >=5% are the appropriate criteria instead
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0092
tools:
- name: RALPS
  role: Produces the normalized data matrix whose replicate cross-correlations are
    evaluated
  repo: https://github.com/zamboni-lab/RALPS
- name: pandas
  role: Loads the data matrix and batch info file and matches replicates via the 'group'
    column
- name: numpy
  role: Computes correlation matrices and aggregates mean replicate cross-correlations
- name: scipy
  role: Pearson correlation computation for replicate cross-correlations
- name: matplotlib
  role: Renders evaluation plots (e.g. mean cross-correlation vs missing-value fraction)
    as png
- name: seaborn
  role: Styling of the cross-correlation evaluation plots
provenance:
  source_task_ids:
  - task_002
  - task_007
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btad096
    title: Regularized adversarial learning for normalization of multi-batch untargeted
      metabolomics data
  source_content_hash: sha256:3c96843a3e01568f33c06b9b1a0439fb3bae4380317e4fbd1e1495eb780bfc4f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/zamboni-lab/RALPS
  tools:
  - PyTorch
  - RALPS
  - hdbscan
  - matplotlib
  - numpy
  - pandas
  - scikit-learn
  - scipy
  - seaborn
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 18
---

# replicate-cross-correlation-evaluation

## Summary

Evaluates multi-batch metabolomics normalization (e.g. by RALPS) by computing mean Pearson cross-correlations among reference/replicate samples across batches, matched via the 'group' column of the batch info file. Higher replicate cross-correlation indicates successful suppression of interbatch differences while preserving biological identity.

## When to use

Use when you have a multi-batch metabolomics feature matrix with replicate reference samples (e.g. pooled or NIST 1950 SRM samples) present in each batch and need to quantify whether a normalization removed batch effects.

## When NOT to use

- The dataset has no replicate reference samples (e.g. pooled QC or NIST 1950 SRM) present across batches, so no cross-batch replicate pairs can be matched via the 'group' column
- You need sample-level biological discrimination rather than batch-effect assessment — benchmark groups are excluded from regularization and only support model evaluation
- Evaluating spectral preservation or variance inflation, where batch-wise VC and fraction of features with VC inflated by >=5% are the appropriate criteria instead

## Inputs

- Normalized data matrix CSV (e.g. normalized/data.csv from RALPS, features x samples)
- Batch info CSV with columns batch, group, benchmark
- Pre-normalization data matrix for baseline replicate cross-correlations
- Missing-value degradation level (fraction of values replaced with 1000 counts) for ablation runs

## Outputs

- Mean replicate cross-correlation per reference group and overall (post_normalization_metrics.csv)
- Comparison of pre- vs post-normalization replicate cross-correlations
- Plots of mean replicate cross-correlation vs missing-value fraction or per batch (matplotlib/seaborn, plots_extension=png)

## How to apply

Load the normalized data matrix (e.g. RALPS output normalized/data.csv with values below min_relevant_intensity=1000 already masked with zeros) and the batch info file with pandas/numpy. Match replicate samples across the seven batches via the 'group' column, excluding benchmark samples that are not used for regularization. Compute pairwise Pearson replicate cross-correlations with scipy/numpy and aggregate the mean per group and overall. Judge success by comparing against pre-normalization metrics: post-normalization replicate cross-correlations should increase and batch-wise coefficients of variation should decrease; for the missing-value robustness ablation, the mean cross-correlation should decline only marginally, from about 0.98 at 0% missing to about 0.94 at 30% missing values replaced with 1000 counts.

## Related tools

- **RALPS** (Produces the normalized data matrix whose replicate cross-correlations are evaluated) — https://github.com/zamboni-lab/RALPS
- **pandas** (Loads the data matrix and batch info file and matches replicates via the 'group' column)
- **numpy** (Computes correlation matrices and aggregates mean replicate cross-correlations)
- **scipy** (Pearson correlation computation for replicate cross-correlations)
- **matplotlib** (Renders evaluation plots (e.g. mean cross-correlation vs missing-value fraction) as png)
- **seaborn** (Styling of the cross-correlation evaluation plots)

## Examples

```
python ralps.py -n path/to/config.csv
```

## Evaluation signals

- Post-normalization replicate cross-correlations are increased and batch-wise coefficients of variation are reduced relative to pre-normalization (Section 2.4 validation criterion)
- Mean replicate cross-correlation at 0% missing values is close to 0.98, declining only marginally to about 0.94 at 30% replaced values (Figure 4d expectation)
- Benchmark samples (flagged in the benchmark column) are excluded from the correlation computation
- All pairs of replicates of the same 'group' across the seven batches are matched; no group is silently dropped
- Normalized values below min_relevant_intensity=1000 are masked with zeros before computing correlations

## Limitations

- Requires replicate reference samples present in every batch (identical across batches, varying between pairs, or carried-over samples); it cannot assess normalization in their absence
- Cross-correlation alone does not capture spectral transformation or variance inflation — pair it with batch VC and fraction of features with VC inflated by >=5% as in the seven-method comparison
- Degraded matrices with >=30% randomly replaced values (1000 counts) push the mean cross-correlation down to ~0.94, so absolute thresholds are sensitive to missing-value fraction

## Evidence

- [other] Compute the mean cross-correlation among replicate samples: [evidence span withheld — anchor status: not_found]
- [methods] accept the result if post-normalization metrics improve: [evidence span withheld — anchor status: not_found]
- [results] 0.98 to 0.94 under missing-value ablation: [evidence span withheld — anchor status: not_found]
- [readme] masking below min_relevant_intensity: [evidence span withheld — anchor status: not_found]
- [abstract] RALPS preserves biological identity: "RALPS builds on deep adversarial learning with a three-term loss function that mitigates batch effects while preserving biological identity, spectral properties and coefficients of variation."
