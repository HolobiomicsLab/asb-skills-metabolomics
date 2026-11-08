---
name: psd-eigenvalue-verification
description: 'Use when a similarity/co-clustering matrix must be certified as a kernel:
  after aggregating H re-sampled co-clustering matrices into a consensus matrix, check
  all eigenvalues are >= 0 before kernel learning.'
when_to_use_negative:
- The input matrix is an arbitrary similarity matrix not built as a weighted sum of
  co-clustering matrices — PSD is not guaranteed and the check may legitimately fail
- You need to construct or weight kernels across datasets (multiple kernel learning)
  rather than certify a single consensus matrix's definiteness
- The matrix is not square/symmetric (e.g. a cluster label assignment or a distance
  matrix with asymmetric entries)
tools:
- name: klic
  role: R package providing the Consensus Clustering implementation whose re-sampled
    co-clustering matrices are aggregated into the consensus matrices being verified
  repo: github.com/acabassi/klic-pancancer-analysis
- name: R
  role: Runtime for eigendecomposition (eigenvalues must all be >= 0) and for writing
    the PSD check results file
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btaa593
    title: Multiple kernel learning for integrative consensus clustering of omic datasets
  source_content_hash: sha256:894a743a4eafca4f3f5f1a8de9e0d257e0597d857ea20531fb42e226992a7e59
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - klic
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# psd-eigenvalue-verification

## Summary

Verifies that a consensus matrix — the non-negative weighted sum of re-sampled co-clustering matrices produced by Consensus Clustering in the klic workflow — is positive semi-definite via eigendecomposition, confirming it can serve as a valid kernel matrix for KLIC.

## When to use

Use when a similarity/co-clustering matrix must be certified as a kernel: after aggregating H re-sampled co-clustering matrices into a consensus matrix, check all eigenvalues are >= 0 before kernel learning.

## When NOT to use

- The input matrix is an arbitrary similarity matrix not built as a weighted sum of co-clustering matrices — PSD is not guaranteed and the check may legitimately fail
- You need to construct or weight kernels across datasets (multiple kernel learning) rather than certify a single consensus matrix's definiteness
- The matrix is not square/symmetric (e.g. a cluster label assignment or a distance matrix with asymmetric entries)

## Inputs

- H re-sampled co-clustering matrices from Consensus Clustering runs per cluster count K (klic R package output)
- Consensus matrices written as consensus_matrix.csv (CSV/NPY) for each K

## Outputs

- Eigendecomposition results (eigenvalues) for each consensus matrix
- PSD check results file recording the minimum eigenvalue per cluster count K
- Verdict per consensus matrix: positive semi-definite (usable as kernel) or not

## How to apply

Run Consensus Clustering (Monti et al. 2003) as implemented in the klic R package over the tested range of cluster counts K, aggregating the re-sampled co-clustering matrices into the consensus matrix (Eq. 1, Sec. 2.1.1) for each K and writing each to consensus_matrix.csv. Perform an eigendecomposition of each consensus matrix and confirm every eigenvalue is >= 0 up to numerical tolerance, which validates the paper's claim (Sec. 2.2.3) that each consensus matrix is PSD because it is a non-negative weighted sum of PSD co-clustering matrices. Record the minimum eigenvalue per K in a PSD check results file alongside the matrices; a negative minimum eigenvalue beyond tolerance signals a numerical or aggregation fault. Only matrices passing the PSD check may be used as kernels in the subsequent Kernel Learning Integrative Clustering (KLIC) step.

## Related tools

- **klic** (R package providing the Consensus Clustering implementation whose re-sampled co-clustering matrices are aggregated into the consensus matrices being verified) — github.com/acabassi/klic-pancancer-analysis
- **R** (Runtime for eigendecomposition (eigenvalues must all be >= 0) and for writing the PSD check results file)

## Examples

```
install.packages("klic"); library(klic); M <- as.matrix(read.csv("consensus_matrix.csv")); ev <- eigen(M, only.values = TRUE)$values; min(ev)  # must be >= -1e-10
```

## Evaluation signals

- All eigenvalues of each consensus matrix are >= 0 up to numerical tolerance (PSD), for every K in the tested range
- Minimum eigenvalue per K is recorded in the PSD check results file next to the saved consensus_matrix.csv files
- Consensus matrices reproduce Eq. (1) of Sec. 2.1.1: a non-negative weighted aggregation of the H re-sampled co-clustering matrices
- PSD-certified matrices are accepted as kernels in the downstream KLIC step; any non-PSD matrix triggers investigation rather than kernel use

## Limitations

- Numerical tolerance is required when checking eigenvalues >= 0; small negative values from floating-point error must not be treated as substantive violations
- The PSD guarantee holds specifically because the consensus matrix is a non-negative weighted sum of PSD co-clustering matrices; modifying the aggregation can break it
- The verification pipeline is tied to the klic/Consensus Clustering (Monti et al. 2003) framework and the pan-cancer reproduction workflow of Cabassi and Kirk (2020)

## Evidence

- [full_text] every consensus matrix is a non-negative weighted sum of positive semi-definite co-clustering matrices, and therefore any consensus matrix is positive semi-definite and usable as a kernel.: [evidence span withheld — anchor status: not_found]
- [methods] For each layer, one kernel is built using Consensus Clustering (Monti et al. 2003) and the most appropriate clustering algorithm.: "For each layer, one kernel is built using Consensus Clustering (Monti et al. 2003) and the most appropriate clustering algorithm."
- [readme] The code contained in this folder can be used to reproduce the pancancer analysis presented in Cabassi and Kirk (2020).: "The code contained in this folder can be used to reproduce the pancancer analysis presented in Cabassi and Kirk (2020)."
- [abstract] R packages klic and coca are available on the Comprehensive R Archive Network.: "R packages klic and coca are available on the Comprehensive R Archive Network."
