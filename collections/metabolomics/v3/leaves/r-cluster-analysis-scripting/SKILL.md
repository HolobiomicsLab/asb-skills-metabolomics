---
name: r-cluster-analysis-scripting
description: Use when per-dataset cluster label vectors (possibly different K_m per
  dataset) must be summarised into a binary cluster-membership matrix before consensus/COCA-style
  integrative clustering.
when_to_use_negative:
- Integrative clustering is done via KLIC instead of COCA — in that pathway kernels
  built by Consensus Clustering (Monti et al. 2003) are the input, not the binary
  MOC
- Input is already a binary cluster-membership/co-association matrix — the MOC construction
  step would be redundant
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: klic
  role: R package (CRAN) for obtaining per-dataset clusterings via consensus clustering
    whose labels feed the MOC
- name: coca
  role: R package (CRAN) implementing Cluster-Of-Clusters Analysis, whose first step
    consumes the MOC built here
- name: R
  role: Scripting environment in which per-dataset cluster labels are loaded, K̄ is
    computed, and the K̄×N binary indicator matrix is assembled and validated
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btaa593
    title: Multiple kernel learning for integrative consensus clustering of omic datasets
  source_content_hash: sha256:98fc36665d34bb0f524c2dda1a9735f9765f83f9055bb3dce6bab0fc981a71b0
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - coca
  - klic
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# r-cluster-analysis-scripting

## Summary

R scripting skill for building the Matrix Of Clusters (MOC) in Cluster-Of-Clusters Analysis (COCA): per-dataset cluster label vectors with differing numbers of clusters are converted into a binary K̄×N indicator matrix that feeds the consensus clustering integration step, using the klic and coca R packages.

## When to use

Use when per-dataset cluster label vectors (possibly different K_m per dataset) must be summarised into a binary cluster-membership matrix before consensus/COCA-style integrative clustering.

## When NOT to use

- Integrative clustering is done via KLIC instead of COCA — in that pathway kernels built by Consensus Clustering (Monti et al. 2003) are the input, not the binary MOC
- Input is already a binary cluster-membership/co-association matrix — the MOC construction step would be redundant

## Inputs

- Per-dataset cluster label vectors c_m (one vector per dataset m, K_m clusters each)
- Number of observations N shared across datasets
- Dataset-missingness pattern (which observations are absent from which dataset)

## Outputs

- Matrix Of Clusters (MOC): K̄×N binary matrix, K̄ = sum of K_m
- moc_matrix.csv (plain CSV, 0/1 entries, no non-binary values)

## How to apply

Load or generate the per-dataset cluster label vectors c_m for M datasets with K_m clusters each (e.g. a small synthetic example, or clusters obtained with the klic/coca packages from CRAN). Compute K̄ = sum of the K_m across datasets, which is the total number of rows of the MOC. For each dataset m and each of its K_m clusters, build a length-N binary indicator row with 1 for observations assigned to that cluster and 0 otherwise, entering zeros for observations missing from that dataset. Stack all K̄ indicator rows into a single K̄×N binary matrix in dataset-then-cluster order and write it out (e.g. moc_matrix.csv) with no non-binary entries. Validate that the file exists, has exactly K̄ rows and N columns, contains only 0/1 entries, and matches a hand-computed reference matrix for the synthetic example.

## Related tools

- **klic** (R package (CRAN) for obtaining per-dataset clusterings via consensus clustering whose labels feed the MOC)
- **coca** (R package (CRAN) implementing Cluster-Of-Clusters Analysis, whose first step consumes the MOC built here)
- **R** (Scripting environment in which per-dataset cluster labels are loaded, K̄ is computed, and the K̄×N binary indicator matrix is assembled and validated)

## Examples

```
install.packages(c("klic", "coca"))
```

## Evaluation signals

- moc_matrix.csv exists and is a plain CSV containing only binary (0/1) entries
- Matrix dimensions are exactly K̄ rows and N columns, where K̄ = sum of the K_m clusters across the M datasets
- Each indicator row has exactly the observations assigned to one cluster of one dataset, with 0 for observations missing from that dataset
- Matrix matches a hand-computed reference MOC for the small synthetic example
- Rows are ordered in dataset-then-cluster order (all clusters of dataset 1, then dataset 2, etc.)

## Limitations

- The MOC enters zeros for observations missing from a dataset, so the encoding implicitly carries dataset-missingness; datasets that are noisy can degrade the downstream COCA consensus clustering
- The properties of COCA were never systematically explored before Cabassi and Kirk (2020) and its robustness to the inclusion of noisy datasets was unclear — KLIC is offered as an alternative that down-weights noisy datasets

## Evidence

- [readme] Integrative clustering is performed using Cluster-Of-Clusters Analysis (Cabassi and Kirk, 2020) and the clusters found in the previous step.: "Integrative clustering is performed using Cluster-Of-Clusters Analysis (Cabassi and Kirk, 2020) and the clusters found in the previous step."
- [abstract] R packages klic and coca are available on the Comprehensive R Archive Network: "R packages klic and coca are available on the Comprehensive R Archive Network."
- [readme] The data used for the analysis are those of Hoadley et al. (2014), which can be downloaded from [Synapse]: "The data used for the analysis are those of Hoadley et al. (2014), which can be downloaded from [Synapse](https://www.synapse.org/#!Synapse:syn2468297/wiki/64259)."
- [readme] Each layer of the dataset is preprocessed in the same way as in Hoadley et al. (2014): "Each layer of the  dataset is preprocessed in the same way as in Hoadley et al. (2014). A clustering of the patient is found in each layer using the same clustering algorithm as Hoadley et al. (2014)."
