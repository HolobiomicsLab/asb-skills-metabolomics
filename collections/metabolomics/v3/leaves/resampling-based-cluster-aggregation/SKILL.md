---
name: resampling-based-cluster-aggregation
description: Use when building kernel matrices from omic data layers via repeated
  resampled base clusterings, or when verifying that a consensus matrix is PSD before
  feeding it to kernel-based integrative clustering.
when_to_use_negative:
- Input is already a valid kernel or similarity matrix — skip consensus clustering
  and go directly to KLIC
- The goal is combining clusterings already computed per layer without kernels — Cluster-Of-Clusters
  Analysis (COCA) is the alternative strategy described in the paper
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: klic
  role: R package providing the Consensus Clustering implementation used to generate
    re-sampled co-clustering matrices and aggregate them into consensus (kernel) matrices
- name: R
  role: Runtime environment in which klic and coca are executed (both available on
    CRAN)
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btaa593
    title: Multiple kernel learning for integrative consensus clustering of omic datasets
  source_content_hash: sha256:c89ccff159d9a80fd8e7a09cf7136b4d6af48df8cd73e15d341c30ee2119d125
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - klic
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# resampling-based-cluster-aggregation

## Summary

Aggregates H re-sampled co-clustering matrices produced by Consensus Clustering (Monti et al. 2003), as implemented in the klic R package, into a consensus matrix (Eq. 1, Sec. 2.1.1) that can be verified as positive semi-definite and reused as a kernel matrix for multiple kernel learning (KLIC).

## When to use

Use when building kernel matrices from omic data layers via repeated resampled base clusterings, or when verifying that a consensus matrix is PSD before feeding it to kernel-based integrative clustering.

## When NOT to use

- Input is already a valid kernel or similarity matrix — skip consensus clustering and go directly to KLIC
- The goal is combining clusterings already computed per layer without kernels — Cluster-Of-Clusters Analysis (COCA) is the alternative strategy described in the paper

## Inputs

- Omic data layer(s) (e.g. gene expression or other patient-by-feature matrices such as the Hoadley et al. 2014 pan-cancer layers)
- Range of cluster counts K to test
- Number of re-samplings H per cluster count
- Cluster labels or a chosen clustering algorithm for the base clusterings

## Outputs

- consensus_matrix.csv (consensus matrix per K, CSV/NPY)
- PSD check results file with minimum eigenvalue per K
- Per-layer kernel matrices ready for KLIC integrative clustering

## How to apply

Install the klic package from CRAN in R and run Consensus Clustering (Monti et al. 2003) over a dataset with known or unknown cluster structure, re-sampling to obtain H base clusterings for each cluster count K in the tested range. Convert each re-sampled clustering into its co-clustering (connectivity) matrix, then aggregate all H matrices as a non-negative weighted sum to form the consensus matrix of Eq. (1) (Sec. 2.1.1) for each K. Because each co-clustering matrix is positive semi-definite, the consensus matrix is itself a non-negative weighted sum of PSD matrices and is therefore PSD (Sec. 2.2.3) and usable as a kernel. Verify this numerically by eigendecomposition, checking all eigenvalues are >= 0 up to numerical tolerance, and record the minimum eigenvalue per K. Write each consensus matrix to consensus_matrix.csv (CSV/NPY) plus a PSD check results file; these kernels then serve as inputs to KLIC in the klic pipeline.

## Related tools

- **klic** (R package providing the Consensus Clustering implementation used to generate re-sampled co-clustering matrices and aggregate them into consensus (kernel) matrices)
- **R** (Runtime environment in which klic and coca are executed (both available on CRAN))

## Examples

```
library(klic); cm <- consensusCluster(data, K = 4, B = 100); eig <- eigen(cm, symmetric = TRUE); min(eig$values) >= -1e-10
```

## Evaluation signals

- All eigenvalues of each consensus matrix are >= 0 up to numerical tolerance (PSD verification of Sec. 2.2.3)
- Minimum eigenvalue per K is recorded and non-negative in the PSD check results file
- Consensus matrix is symmetric with unit diagonal and entries bounded in [0, 1], as a weighted sum of co-clustering matrices
- One consensus matrix is produced per tested cluster count K in the range specified
- Downstream KLIC run accepts the consensus matrices as kernels without validation errors

## Limitations

- PSD holds only up to numerical tolerance in eigendecomposition; tiny negative eigenvalues from floating-point error may appear
- The choice of clustering algorithm per layer must be matched to the data ('the most appropriate clustering algorithm')
- The klic-pancancer-analysis code depends on the Hoadley et al. (2014) Synapse data (syn2468297), which requires separate download

## Evidence

- [methods] For each layer, one kernel is built using Consensus Clustering (Monti et al. 2003) and the most appropriate clustering algorithm.: "For each layer, one kernel is built using Consensus Clustering (Monti et al. 2003) and the most appropriate clustering algorithm."
- [abstract] R packages klic and coca are available on the Comprehensive R Archive Network: "R packages klic and coca are available on the Comprehensive R Archive Network"
- [readme] The code contained in this folder can be used to reproduce the pancancer analysis presented in Cabassi and Kirk (2020).: "The code contained in this folder can be used to reproduce the pancancer analysis presented in Cabassi and Kirk (2020)."
- [readme] The data used for the analysis are those of Hoadley et al. (2014), which can be downloaded from [Synapse](https://www.synapse.org/#!Synapse:syn2468297/wiki/64259).: "The data used for the analysis are those of Hoadley et al. (2014), which can be downloaded from [Synapse](https://www.synapse.org/#!Synapse:syn2468297/wiki/64259)."
