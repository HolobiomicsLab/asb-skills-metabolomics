---
name: cluster-membership-matrix-construction
description: Use when you have per-dataset cluster label vectors c_m (M omic datasets,
  K_m clusters each) and need a single binary matrix encoding cluster memberships
  as input to COCA consensus clustering.
when_to_use_negative:
- No per-dataset hard clusterings exist yet and you plan to use KLIC instead, where
  each layer is integrated via kernels built with Consensus Clustering (Monti et al.
  2003) rather than a cluster-membership matrix
- Input is already a similarity/kernel matrix or the final COCA consensus clustering
  output — this skill only produces the intermediate binary MOC
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0091
tools:
- name: klic
  role: R package (CRAN) used to load or generate the per-dataset cluster label vectors
    c_m that feed the MOC
- name: coca
  role: R package (CRAN) for Cluster-Of-Clusters Analysis, the integrative clustering
    step that consumes the constructed MOC
- name: R
  role: Environment in which the MOC is built and written to moc_matrix.csv and in
    which the pan-cancer analysis scripts are run
  repo: github.com/acabassi/klic-pancancer-analysis
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btaa593
    title: Multiple kernel learning for integrative consensus clustering of omic datasets
  source_content_hash: sha256:f96e53b7c2c2508de9e430f388efd375810670a96e55c849418a7e2a07fe9b7f
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - coca
  - klic
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# cluster-membership-matrix-construction

## Summary

Constructs the Matrix Of Clusters (MOC) used by Cluster-Of-Clusters Analysis (COCA): a K̄×N binary matrix that stacks one indicator row per cluster from every dataset, where K̄ is the sum of the K_m clusters across the M datasets. This is the intermediate representation that lets per-dataset clusterings with different numbers of clusters be integrated by a single consensus clustering step.

## When to use

Use when you have per-dataset cluster label vectors c_m (M omic datasets, K_m clusters each) and need a single binary matrix encoding cluster memberships as input to COCA consensus clustering.

## When NOT to use

- No per-dataset hard clusterings exist yet and you plan to use KLIC instead, where each layer is integrated via kernels built with Consensus Clustering (Monti et al. 2003) rather than a cluster-membership matrix
- Input is already a similarity/kernel matrix or the final COCA consensus clustering output — this skill only produces the intermediate binary MOC

## Inputs

- Per-dataset cluster label vectors c_m for M datasets, each with K_m clusters (e.g. from klic package clustering or preprocessing scripts)
- N, the number of shared observations (e.g. patients) common to the datasets

## Outputs

- moc_matrix.csv: a K̄×N binary (0/1) CSV matrix, rows ordered dataset-then-cluster, encoding each observation's cluster membership in each dataset

## How to apply

Load or generate the per-dataset cluster label vectors c_m, either from a small synthetic example or via the klic R package from CRAN. Compute K̄ = sum over datasets of K_m, the total number of rows of the MOC. For each dataset m and each of its K_m clusters, build a length-N binary indicator row with 1 for observations assigned to that cluster and 0 otherwise, entering zeros for observations missing from that dataset. Stack all K̄ indicator rows into a K̄×N binary matrix in dataset-then-cluster order and write it to moc_matrix.csv in plain CSV format with no non-binary entries. In the pan-cancer pipeline of Cabassi and Kirk (2020), this MOC is what the subsequent COCA integrative clustering step consumes after per-layer preprocessing and clustering (performed as in Hoadley et al. 2014).

## Related tools

- **klic** (R package (CRAN) used to load or generate the per-dataset cluster label vectors c_m that feed the MOC)
- **coca** (R package (CRAN) for Cluster-Of-Clusters Analysis, the integrative clustering step that consumes the constructed MOC)
- **R** (Environment in which the MOC is built and written to moc_matrix.csv and in which the pan-cancer analysis scripts are run) — github.com/acabassi/klic-pancancer-analysis

## Examples

```
Rscript -e "install.packages(c('klic','coca')); library(coca); # build K_bar x N binary MOC from per-dataset cluster labels c_m and write.csv(moc_matrix, 'moc_matrix.csv', row.names = FALSE)"
```

## Evaluation signals

- moc_matrix.csv exists and has exactly K̄ rows (= sum of K_m across the M datasets) and N columns
- Every entry of the written CSV is binary (0 or 1) — no non-binary values
- Rows are stacked in dataset-then-cluster order, and rows for observations missing from a dataset m are 0 across that dataset's K_m rows
- The matrix matches a hand-computed reference MOC for a small synthetic example with known cluster labels
- Downstream COCA run on the MOC yields an integrative consensus clustering consistent with the source paper's pan-cancer workflow

## Limitations

- The MOC grows as K̄×N, so datasets with many clusters inflate the number of indicator rows that the consensus clustering must integrate
- Zeros encode both non-membership and missingness (observations absent from a dataset get zeros), which can be confounded
- COCA's robustness to including noisy datasets was unclear prior to systematic exploration in Cabassi and Kirk (2020), which motivates the KLIC alternative with weighted per-dataset contributions

## Evidence

- [methods] coca methods step: "Integrative clustering is performed using Cluster-Of-Clusters Analysis (Cabassi and Kirk, 2020) and the clusters found in the previous step."
- [methods] per-layer clustering prerequisite: "For each layer, one kernel is built using Consensus Clustering (Monti et al. 2003) and the most appropriate clustering algorithm."
- [readme] readme folder structure: "The code contained in this folder can be used to reproduce the pancancer analysis presented in Cabassi and Kirk (2020)."
- [abstract] tool availability: "R packages klic and coca are available on the Comprehensive R Archive Network"
