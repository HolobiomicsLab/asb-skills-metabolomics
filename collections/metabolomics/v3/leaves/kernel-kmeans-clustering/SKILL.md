---
name: kernel-kmeans-clustering
description: Use when samples are represented by one or more N x N similarity/kernel
  matrices (e.g. consensus matrices built with Monti et al. 2003) rather than raw
  feature tables, and you need cluster assignments, optionally with per-dataset or
  per-sample kernel weights.
when_to_use_negative:
- Input is a raw feature matrix with no consensus/kernel matrices built — build kernels
  first via the Consensus Clustering (Kernel) step of the pipeline
- Datasets lack a shared clustering structure or sample alignment, so convex combinations
  of kernels are not meaningful
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3307
tools:
- name: klic
  role: R package providing kernel k-means and KLIC (Kernel Learning Integrative Clustering)
    with multiple kernel learning weights over the built kernels
  repo: https://www.synapse.org/#!Synapse:syn2468297/wiki/64259
- name: R
  role: Implementation language for the simulation, kernel construction, and kernel
    k-means baselines
provenance:
  source_task_ids:
  - task_002
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btaa593
    title: Multiple kernel learning for integrative consensus clustering of omic datasets
  source_content_hash: sha256:57a9379f53761c3707a77255d2ba17479767ad858a82957fdcb39174b531ce49
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - coca
  - klic
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3307
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8461538461538461
n_claims: 13
---

# kernel-kmeans-clustering

## Summary

Kernel k-means clustering on similarity (consensus) kernel matrices, used both as a single-dataset baseline and as the clustering engine inside KLIC's multiple kernel learning framework (Cabassi and Kirk 2020), optionally with per-sample kernel weights.

## When to use

Use when samples are represented by one or more N x N similarity/kernel matrices (e.g. consensus matrices built with Monti et al. 2003) rather than raw feature tables, and you need cluster assignments, optionally with per-dataset or per-sample kernel weights.

## When NOT to use

- Input is a raw feature matrix with no consensus/kernel matrices built — build kernels first via the Consensus Clustering (Kernel) step of the pipeline
- Datasets lack a shared clustering structure or sample alignment, so convex combinations of kernels are not meaningful

## Inputs

- One or more N x N similarity (consensus) kernel matrices over the same N samples
- True cluster labels for synthetic benchmark data (for ARI validation)
- Number of clusters k

## Outputs

- Cluster assignments per sample
- N x M matrix of sample-specific kernel weights (each row summing to 1), saved as mkkm_model.rds
- Adjusted Rand index (ARI) against true labels
- Per-dataset kernel weights averaged over repetitions (simulation_ari_kernel_weights.csv)

## How to apply

Build one N x N kernel per dataset using Consensus Clustering (Monti et al. 2003) with the most appropriate base clustering algorithm, as in the Kernel step of the KLIC pipeline. Run kernel k-means on a single kernel as a single-dataset baseline, or on the convexly combined kernel sum_m theta_m K_m for integrative clustering; in the localized multiple-kernel variant, alternate (i) kernel k-means on the combined kernel with fixed weights and (ii) a convex quadratic program updating the sample-specific weights theta_im (each row summing to 1 over M kernels) until assignments and weights converge. Evaluate clusters against true labels with the adjusted Rand index (ARI), and for KLIC record the kernel weights, which down-weight noisy layers relative to more informative ones. In the noise-levels simulation (separability levels 0-3, 100 repetitions), the dataset with the clearest clustering structure receives the highest average kernel weight while separability level 0 receives the smallest.

## Related tools

- **klic** (R package providing kernel k-means and KLIC (Kernel Learning Integrative Clustering) with multiple kernel learning weights over the built kernels) — https://www.synapse.org/#!Synapse:syn2468297/wiki/64259
- **R** (Implementation language for the simulation, kernel construction, and kernel k-means baselines)

## Examples

```
library(klic); fit <- kkmeans(K, centers = 4)  # kernel k-means on a consensus matrix K with k = 4 clusters
```

## Evaluation signals

- ARI of kernel k-means assignments against true labels exceeds the corresponding single-kernel baseline and lies in a plausible range
- Each row of the sample-specific weight matrix is a valid convex weight vector summing to 1 over the M kernels
- In three-way KLIC combinations of separability levels 0-3, the highest-separability dataset receives the highest average kernel weight and level 0 the smallest
- ARI increases as more informative (higher-separability) layers are included in the combination
- The two-step alternation (kernel k-means then convex QP weight update) converges: assignments and weights stabilise across iterations

## Limitations

- Kernel weights down-weight noisy datasets only when the shared clustering structure differs in separability; with equally informative layers, weights are not informative
- Requires consensus matrices built with an appropriately chosen base clustering algorithm per layer; poor kernel choice degrades clustering
- Localized multiple kernel k-means weight updates are a convex QP per sample but the overall alternating scheme guarantees only local convergence

## Evidence

- [readme] For each layer, one kernel is built using Consensus Clustering (Monti et al. 2003) and the most appropriate clustering algorithm.: "For each layer, one kernel is built using Consensus Clustering (Monti et al. 2003) and the most appropriate clustering algorithm."
- [methods] KLIC step defined on kernels: "Integrative clustering is performed using Kernel Learning Integrative Clustering (Cabassi and Kirk, 2020) and the kernels built in the previous step."
- [abstract] Noisy datasets down-weighted: "This allows the contribution of noisy datasets to be down-weighted relative to more informative datasets"
