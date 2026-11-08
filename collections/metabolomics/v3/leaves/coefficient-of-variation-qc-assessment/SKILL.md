---
name: coefficient-of-variation-qc-assessment
description: Use when a multi-batch metabolomics feature-count matrix has been normalized
  (e.g. by RALPS) and you need QC metrics — batch-wise VC and replicate cross-correlation
  — to judge whether batch effects were reduced.
when_to_use_negative:
- Data is raw and not yet batch-normalized — intrabatch drifts should first be removed
  by normalizing each batch individually before computing cross-batch VC metrics
- Data contains no replicate or reference sample groups (e.g. NIST 1950 SRM or pooled
  samples) present across batches, so replicate cross-correlation and within-batch-replicate
  VC cannot be defined
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: RALPS
  role: normalization method whose output matrix is evaluated with batch VC and replicate
    cross-correlation metrics
  repo: github.com/zamboni-lab/RALPS
- name: pandas
  role: loading the normalized data and batch info files and aggregating per-scenario
    metric values
  repo: github.com/dmitrav/normalization
- name: numpy
  role: computing coefficients of variation and cross-correlations on the normalized
    matrix
- name: scikit-learn
  role: supporting clustering-based evaluation criterion on benchmark samples
- name: hdbscan
  role: clustering benchmark samples (with UMAP) for the clustering evaluation criterion
    alongside VC and cross-correlation
- name: matplotlib
  role: rendering evaluation plots of VC and cross-correlation results
- name: seaborn
  role: rendering evaluation plots of VC and cross-correlation results
provenance:
  source_task_ids:
  - task_005
  - task_002
  - task_006
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btad096
    title: Regularized adversarial learning for normalization of multi-batch untargeted
      metabolomics data
  source_content_hash: sha256:d702ca0515fec2410b90014ef8875d12e1223016f6da35e2dc1c287c215b62b2
schema_version: 0.2.0
metadata:
  license_tier: open
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
  - umap-learn
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4666666666666667
n_claims: 15
---

# coefficient-of-variation-qc-assessment

## Summary

Assess normalization quality of multi-batch metabolomics data by computing replicate cross-correlations and batch-wise coefficients of variation (VC) of the normalized data, the primary QC metrics used by RALPS alongside clustering criteria.

## When to use

Use when a multi-batch metabolomics feature-count matrix has been normalized (e.g. by RALPS) and you need QC metrics — batch-wise VC and replicate cross-correlation — to judge whether batch effects were reduced.

## When NOT to use

- Data is raw and not yet batch-normalized — intrabatch drifts should first be removed by normalizing each batch individually before computing cross-batch VC metrics
- Data contains no replicate or reference sample groups (e.g. NIST 1950 SRM or pooled samples) present across batches, so replicate cross-correlation and within-batch-replicate VC cannot be defined

## Inputs

- normalized feature-count CSV matrix (e.g. normalized/data.csv), with values below min_relevant_intensity=1000 masked as zeros
- batch info CSV file with batch, group and benchmark columns identifying replicate/reference groups
- optionally pre-normalization data matrix for before/after comparison

## Outputs

- post_normalization_metrics.csv with replicate cross-correlation and batch VC results
- evaluation plots (png) of replicate cross-correlation and batch-wise VC rendered with matplotlib/seaborn
- mean batch VC values for scenario/ablation comparisons

## How to apply

After running RALPS normalization (`python ralps.py -n path/to/config.csv`), load the normalized matrix (normalized/data.csv) and the batch info file (batch, group, benchmark columns) with pandas, treating replicate reference groups (e.g. undiluted NIST1950 serum extract P2_S_0001 and purine/pyrimidine-spiked serum P2_S_PP_0001) as replicate measurements of the same material. Compute replicate cross-correlations across the seven batches and batch-wise coefficients of variation of the normalized data, using scikit-learn/hdbscan clustering if adding the clustering criterion. Note that RALPS masks post-normalization values below min_relevant_intensity=1000 with zeros, which should be respected when computing metrics. Aggregate per-scenario or per-method metric values with pandas and render comparisons with matplotlib/seaborn. Accept the normalization if post-normalization replicate cross-correlations increase and batch-wise coefficients of variation decrease relative to pre-normalization.

## Related tools

- **RALPS** (normalization method whose output matrix is evaluated with batch VC and replicate cross-correlation metrics) — github.com/zamboni-lab/RALPS
- **pandas** (loading the normalized data and batch info files and aggregating per-scenario metric values) — github.com/dmitrav/normalization
- **numpy** (computing coefficients of variation and cross-correlations on the normalized matrix)
- **scikit-learn** (supporting clustering-based evaluation criterion on benchmark samples)
- **hdbscan** (clustering benchmark samples (with UMAP) for the clustering evaluation criterion alongside VC and cross-correlation)
- **matplotlib** (rendering evaluation plots of VC and cross-correlation results)
- **seaborn** (rendering evaluation plots of VC and cross-correlation results)

## Examples

```
python ralps.py -n path/to/config.csv
```

## Evaluation signals

- Post-normalization replicate cross-correlations across the seven batches are increased relative to pre-normalization values
- Batch-wise coefficients of variation of the normalized data are reduced relative to pre-normalization values
- When comparing loss-term ablation scenarios, the scenario with all three non-zero coefficients (d_lambda, g_lambda, v_lambda) attains top performance on the clustering, cross-correlation and VC criteria
- Mean batch VC stays near-constant during metabolite-wise shrinkage down to ~10% of features (about 17 metabolites), worsening tangibly only at the last ablation point
- Normalized data no longer shows batches as isolated clusters — samples from different batches are mostly well mixed

## Limitations

- Values below min_relevant_intensity=1000 are masked with zeros after normalization, which must be accounted for when computing VCs to avoid underestimating variability
- VC-based evaluation requires replicate reference samples (e.g. pooled study samples or NIST 1950 SRM) present in every batch; without them, interbatch differences cannot be assessed
- Per-sample VC increases beyond allowed_vc_increase=0.05 are penalized during RALPS training, so VC behavior is partly an optimization target rather than a fully independent check

## Evidence

- [readme] missing values before normalization are replaced with this;values below this after normalization are masked with zeros: [evidence span withheld — anchor status: not_found]
- [readme] RALPS exploits reference samples present in every batch to assess interbatch differences: "RALPS exploits reference samples (e.g. pooled study samples, NIST 1950 SRM, you name it...) present in every batch to assess interbatch differences."
- [results] batch-wise coefficients of variation of the normalized data: [evidence span withheld — anchor status: not_found]
- [methods] variation loss VC increase allowed fraction: "fraction of sample's VC increase allowed (not contributing to the variation loss)"
- [abstract] three-term loss preserves coefficients of variation: "RALPS builds on deep adversarial learning with a three-term loss function that mitigates batch effects while preserving biological identity, spectral properties and coefficients of variation."
