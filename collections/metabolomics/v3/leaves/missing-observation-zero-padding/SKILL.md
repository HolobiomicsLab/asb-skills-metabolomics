---
name: missing-observation-zero-padding
description: Use when per-dataset clusterings (label vectors c_m with K_m clusters
  each) cover different subsets of N observations and must be stacked into a single
  binary K̄×N matrix before Cluster-Of-Clusters Analysis.
when_to_use_negative:
- Input clusterings already share the same fully observed sample set and a similarity/kernel-based
  integrative method such as KLIC is preferred, since KLIC down-weights noisy datasets
  instead of zero-padding missing entries
- The per-dataset outputs are continuous (e.g. kernel/similarity matrices from consensus
  clustering) rather than discrete cluster labels — the MOC encoding requires binary
  membership vectors
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: klic
  role: R package supplying the per-dataset consensus clusterings and MOC machinery
    used to build the cluster-of-clusters matrix
  repo: https://github.com/acabassi/klic-pancancer-analysis
- name: coca
  role: R package implementing Cluster-Of-Clusters Analysis, which consumes the zero-padded
    MOC for the consensus clustering integration step
  repo: https://github.com/acabassi/klic-pancancer-analysis
- name: R
  role: Environment in which the klic and coca packages are run to construct the MOC
    from cluster label vectors
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btaa593
    title: Multiple kernel learning for integrative consensus clustering of omic datasets
  source_content_hash: sha256:1ce4bc11b0cfbec7ff58387297103251b3594edd5042fd059e34812f1d780f0f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/acabassi/klic-pancancer-analysis
  tools:
  - R
  - coca
  - klic
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# missing-observation-zero-padding

## Summary

Encodes per-dataset cluster assignments of partially observed samples into COCA's binary Matrix Of Clusters (MOC) of size K̄×N, entering zeros for observations missing from a dataset so that consensus clustering can proceed on the union of all samples.

## When to use

Use when per-dataset clusterings (label vectors c_m with K_m clusters each) cover different subsets of N observations and must be stacked into a single binary K̄×N matrix before Cluster-Of-Clusters Analysis.

## When NOT to use

- Input clusterings already share the same fully observed sample set and a similarity/kernel-based integrative method such as KLIC is preferred, since KLIC down-weights noisy datasets instead of zero-padding missing entries
- The per-dataset outputs are continuous (e.g. kernel/similarity matrices from consensus clustering) rather than discrete cluster labels — the MOC encoding requires binary membership vectors

## Inputs

- Per-dataset cluster label vectors c_m for M datasets, each with K_m clusters (e.g. from consensus clustering in the klic package)
- N, the total number of observations across all datasets (e.g. patients in the Hoadley et al. 2014 pan-cancer data)

## Outputs

- moc_matrix.csv: a K̄×N binary CSV matrix (K̄ = sum of K_m) where each row is a cluster-membership indicator and missing observations are 0

## How to apply

Load the per-dataset cluster label vectors c_m for M datasets and compute K̄ = sum_m K_m, the total number of rows of the MOC. For each dataset m and each of its K_m clusters, build a length-N binary indicator row with 1 for observations assigned to that cluster and 0 otherwise; where an observation is missing from dataset m, enter 0 in all of that dataset's indicator rows rather than dropping the sample. Stack all K̄ indicator rows in dataset-then-cluster order into a single K̄×N binary matrix and write it as plain CSV (moc_matrix.csv). This zero-padding is what allows COCA's consensus clustering step to integrate clusterings with different K_m and different observed sample sets into one unified clustering of all N observations. The klic and coca R packages (Cabassi and Kirk, 2020, Bioinformatics) implement this machinery.

## Related tools

- **klic** (R package supplying the per-dataset consensus clusterings and MOC machinery used to build the cluster-of-clusters matrix) — https://github.com/acabassi/klic-pancancer-analysis
- **coca** (R package implementing Cluster-Of-Clusters Analysis, which consumes the zero-padded MOC for the consensus clustering integration step) — https://github.com/acabassi/klic-pancancer-analysis
- **R** (Environment in which the klic and coca packages are run to construct the MOC from cluster label vectors)

## Examples

```
Rscript -e 'library(coca); moc <- matrix(0, nrow=K_bar, ncol=N); write.csv(moc, "moc_matrix.csv", row.names=FALSE)'
```

## Evaluation signals

- moc_matrix.csv exists and has exactly K̄ rows (= sum_m K_m) and N columns
- All entries of the written matrix are binary (0/1 only); no non-binary or NA values
- Matrix matches the hand-computed reference MOC for a small synthetic example with known labels and known missing observations
- Rows appear in dataset-then-cluster order, and each observation missing from dataset m has zeros in all K_m of that dataset's rows
- Each cluster indicator row sums to the size of the corresponding cluster in dataset m (not N), confirming zero-padding of missing samples

## Limitations

- Zero-padding treats a missing observation as 'not in any cluster' of that dataset, which conflates missingness with non-membership; the article notes COCA's robustness (e.g. to noisy or partially informative datasets) was unclear before its systematic exploration
- The MOC grows linearly with the total number of clusters K̄ across datasets, which can become large for many datasets with many clusters each

## Evidence

- [other] COCA summarises the independently obtained per-dataset clusterings into a Matrix Of Clusters (MOC) of size K̄×N: [evidence span withheld — anchor status: not_found]
- [other] build a length-N binary indicator row with 1 for observations assigned to that cluster and 0 otherwise, entering zeros for observations missing: [evidence span withheld — anchor status: not_found]
- [readme] Integrative clustering is performed using Cluster-Of-Clusters Analysis (Cabassi and Kirk, 2020) and the clusters found in the previous step.: "**COCA**. Integrative clustering is performed using Cluster-Of-Clusters Analysis (Cabassi and Kirk, 2020) and the clusters found in the previous step."
- [abstract] R packages klic and coca are available on the Comprehensive R Archive Network.: "R packages klic and coca are available on the Comprehensive R Archive Network."
- [abstract] the properties of COCA have never been systematically explored, and its robustness to the inclusion of noisy datasets is unclear: "the properties of COCA have never been systematically explored, and its robustness to the inclusion of noisy datasets is unclear"
