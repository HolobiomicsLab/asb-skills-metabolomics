---
name: consensus-clustering-kernel-construction
description: Use when integrating multi-omics datasets (e.g. copy number, methylation,
  mRNA, miRNA, protein layers) for clustering and per-layer similarity kernels are
  needed as inputs to KLIC or MKL.
when_to_use_negative:
- The input is already a valid similarity/kernel matrix or a feature table destined
  for a non-kernel method such as plain iCluster or Clusternomics, which take raw
  data rather than consensus kernels
- The datasets do not share the same set of observations, since the per-layer kernels
  must be combined on a common sample index by KLIC
- Only a single dataset with no integration goal is needed — plain consensus clustering
  or k-means on the consensus matrix suffices without kernel learning
edam_operation: http://edamontology.org/operation_0362
tools:
- name: klic
  role: R package implementing Consensus Clustering (Monti et al. 2003) for building
    per-layer consensus-matrix kernels, and Kernel Learning Integrative Clustering
    that consumes them
  repo: github.com/acabassi/klic-pancancer-analysis
- name: R
  role: Runtime environment; klic and coca are available on CRAN
provenance:
  source_task_ids:
  - task_002
  - task_003
  - task_004
  - task_007
  - task_008
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btaa593
    title: Multiple kernel learning for integrative consensus clustering of omic datasets
  source_content_hash: sha256:e4aa94e10470b8e59bfa3e835697ea97adae3cc58f42f114b7585084e773f562
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - coca
  - klic
  edam_operation: http://edamontology.org/operation_0362
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# consensus-clustering-kernel-construction

## Summary

Constructs per-dataset similarity kernels by aggregating re-sampled co-clustering matrices from Consensus Clustering (Monti et al. 2003) as implemented in the klic R package, so that each omic layer can be fed into multiple kernel learning (KLIC).

## When to use

Use when integrating multi-omics datasets (e.g. copy number, methylation, mRNA, miRNA, protein layers) for clustering and per-layer similarity kernels are needed as inputs to KLIC or MKL.

## When NOT to use

- The input is already a valid similarity/kernel matrix or a feature table destined for a non-kernel method such as plain iCluster or Clusternomics, which take raw data rather than consensus kernels
- The datasets do not share the same set of observations, since the per-layer kernels must be combined on a common sample index by KLIC
- Only a single dataset with no integration goal is needed — plain consensus clustering or k-means on the consensus matrix suffices without kernel learning

## Inputs

- Multi-layer omic data matrices with observations on a common sample set (e.g. Hoadley et al. 2014 pan-cancer layers: DNA copy number, methylation, mRNA, miRNA, protein)
- Choice of base clustering algorithm per layer (e.g. PAM, BHC, k-means) and candidate cluster counts K
- Synthetic datasets with known cluster labels for validation (e.g. 300 observations, six equal clusters, separability levels 0-3)

## Outputs

- One consensus matrix per layer per K (consensus_matrix.csv / NPY), usable as a PSD kernel matrix
- PSD check file with minimum eigenvalue per K from eigendecomposition
- Per-dataset kernel weight table (klic_kernel_weights.csv) produced downstream when the kernels are combined with KLIC

## How to apply

For each data layer, run Consensus Clustering (Monti et al. 2003) via the klic R package, selecting the most appropriate base clustering algorithm for that layer (e.g. PAM for expression data, BHC for ChIP-chip data, and per-layer algorithms as in Hoadley et al. 2014 for the pan-cancer data). Aggregate the re-sampled co-clustering matrices into a consensus matrix for each tested cluster count K. Because every consensus matrix is a non-negative weighted sum of positive semi-definite co-clustering matrices, verify via eigendecomposition that all eigenvalues are >= 0 (up to numerical tolerance) so the matrix is a valid kernel. Write the consensus matrices (e.g. consensus_matrix.csv) and PSD check results (minimum eigenvalue per K) before passing the kernels to KLIC.

## Related tools

- **klic** (R package implementing Consensus Clustering (Monti et al. 2003) for building per-layer consensus-matrix kernels, and Kernel Learning Integrative Clustering that consumes them) — github.com/acabassi/klic-pancancer-analysis
- **R** (Runtime environment; klic and coca are available on CRAN)

## Evaluation signals

- Eigendecomposition of each consensus matrix yields minimum eigenvalue >= 0 up to numerical tolerance, confirming positive semi-definiteness
- In the noise-levels simulation, the highest-separability dataset receives the largest average KLIC kernel weight and the separability-0 (noisy) layer the smallest in each three-way subset
- For four similar datasets, average kernel weights are divided approximately equally across datasets A, B, C, D, and combined KLIC ARI exceeds single-dataset kernel k-means ARI
- On the Hoadley et al. (2014) data, per-dataset weights match copy number 31.4%, methylation 19.2%, miRNA 17.8%, mRNA 16.4%, protein 15.2% within rounding tolerance

## Limitations

- The most appropriate base clustering algorithm must be chosen per layer (e.g. PAM vs BHC); a poor base algorithm degrades the consensus kernel
- The ChIP-chip kernel in the yeast analysis was noisier and required down-weighting by KLIC, showing that not all consensus kernels are equally informative
- PSD validity holds only for matrices aggregated as non-negative weighted sums of co-clustering matrices; numerical tolerance is needed in eigendecomposition

## Evidence

- [readme] For each layer, one kernel is built using Consensus Clustering (Monti et al. 2003) and the most appropriate clustering algorithm.: "For each layer, one kernel is built using Consensus Clustering (Monti et al. 2003) and the most appropriate clustering algorithm."
- [readme] Integrative clustering is performed using Kernel Learning Integrative Clustering (Cabassi and Kirk, 2020) and the kernels built in the previous step.: "Integrative clustering is performed using Kernel Learning Integrative Clustering (Cabassi and Kirk, 2020) and the kernels built in the previous step."
- [abstract] R packages klic and coca are available on the Comprehensive R Archive Network: "R packages klic and coca are available on the Comprehensive R Archive Network"
