---
name: multiple-kernel-learning-clustering
description: Use when several omic data layers over the same N samples have been converted
  to per-layer N x N kernel/consensus matrices and you need an integrative clustering
  whose per-dataset weights reveal which layers carry the cluster structure.
when_to_use_negative:
- Input is a raw feature/data matrix rather than per-layer kernel or consensus matrices
  — first run the Kernel step (Consensus Clustering per layer) before applying KLIC.
- Only one dataset/layer is available — combining kernels is unnecessary; run plain
  kernel k-means on the single consensus matrix instead.
- Samples are not matched across the layers (each layer must be an N x N kernel over
  the same N samples) or the layers do not share a common clustering structure.
edam_operation: http://edamontology.org/operation_3432
edam_topics: []
tools:
- name: klic
  role: Implements Consensus Clustering kernel construction and the KLIC/localized
    multiple kernel k-means optimisation; available on CRAN.
- name: coca
  role: Provides the Cluster-Of-Clusters Analysis (COCA) baseline against which KLIC's
    ARI is compared.
- name: R
  role: Language in which the simulation pipelines, benchmark scripts, and ARI aggregation
    are implemented.
  repo: github.com/acabassi/klic-pancancer-analysis
provenance:
  source_task_ids:
  - task_003
  - task_005
  - task_001
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btaa593
    title: Multiple kernel learning for integrative consensus clustering of omic datasets
  source_content_hash: sha256:6edb07775ee6640bea79a1689e1da137816c21f208e0eb5a3b01ae5c9ba77f72
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - coca
  - klic
  edam_operation: http://edamontology.org/operation_3432
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# multiple-kernel-learning-clustering

## Summary

Integrative clustering via Kernel Learning Integrative Clustering (KLIC): combine per-dataset kernel (consensus) matrices by learning sample-specific convex kernel weights with localized multiple kernel k-means, so that noisy datasets are down-weighted relative to more informative ones.

## When to use

Use when several omic data layers over the same N samples have been converted to per-layer N x N kernel/consensus matrices and you need an integrative clustering whose per-dataset weights reveal which layers carry the cluster structure.

## When NOT to use

- Input is a raw feature/data matrix rather than per-layer kernel or consensus matrices — first run the Kernel step (Consensus Clustering per layer) before applying KLIC.
- Only one dataset/layer is available — combining kernels is unnecessary; run plain kernel k-means on the single consensus matrix instead.
- Samples are not matched across the layers (each layer must be an N x N kernel over the same N samples) or the layers do not share a common clustering structure.

## Inputs

- M N x N kernel/consensus matrices over the same N samples, one per omic layer (built via Consensus Clustering, Monti et al. 2003)
- Number of clusters K for the integrative clustering
- True cluster labels (for simulation validation via ARI)

## Outputs

- Cluster assignments for the N samples from the KLIC/localized MK k-means fit
- N x M sample-specific kernel weight matrix (each row a convex weight vector summing to 1 over the M kernels)
- ARI of the clustering against true labels, and aggregated tables of median ARI per method per simulation setting (e.g. CSV of ARI and average per-dataset kernel weights over repetitions)

## How to apply

First build one N x N kernel per data layer using Consensus Clustering (Monti et al. 2003) with the most appropriate clustering algorithm for each layer. Then run localized multiple kernel k-means from the klic R package with the chosen number of clusters K (e.g. K = 6 in the six-cluster simulations): initialise sample-specific weights theta_im with sum_m theta_im = 1 for each sample, alternate (1) kernel k-means on the convex combination sum_i theta_im K_m and (2) a convex quadratic program that re-optimises theta_im with assignments fixed, until assignments and weights converge. Compare against single-layer kernel k-means baselines and the COCA baseline (k-means followed by hierarchical clustering with average linkage) using the adjusted Rand index (ARI) against true labels, aggregating median ARI over ~100 simulation repetitions. Interpret the N x M weight matrix: layers with clearer cluster structure (higher separability) should receive larger average weights, and the noisiest layer the smallest. For kernel-based benchmarks, include both an RBF-MKL variant with sigma fixed to 1 and one with sigma tuned on true labels, since the tuning choice changes which method ranks best.

## Related tools

- **klic** (Implements Consensus Clustering kernel construction and the KLIC/localized multiple kernel k-means optimisation; available on CRAN.)
- **coca** (Provides the Cluster-Of-Clusters Analysis (COCA) baseline against which KLIC's ARI is compared.)
- **R** (Language in which the simulation pipelines, benchmark scripts, and ARI aggregation are implemented.) — github.com/acabassi/klic-pancancer-analysis

## Examples

```
library(klic); fit <- klic(kernel_matrices, K = 6)
```

## Evaluation signals

- Each row of the learned N x M weight matrix is a valid convex weight vector (weights are non-negative and sum to 1 over the M kernels).
- ARI of the KLIC clustering against true labels exceeds the single-dataset kernel k-means baselines (e.g. combining four similar datasets improves ARI over clustering each consensus matrix alone).
- Average kernel weight is largest for the layer with the clearest structure and smallest for the noisiest layer in each multi-dataset combination.
- Median-ARI rankings reproduce the benchmark pattern: RBF-MKL best on similar datasets, COCA best with noisy variables, KLIC and tuned-RBF MKL best with noisy datasets.
- The alternating optimisation converges: cluster assignments and weights stabilise between the kernel k-means step and the convex QP weight-update step.

## Limitations

- The tuned localized multiple kernel k-means needs the RBF sigma parameter tuned (e.g. on true labels in benchmarks); with sigma fixed to 1 it is not always the top performer, so ranking claims depend on that tuning choice.
- KLIC is not uniformly superior to COCA: COCA performs best in the noisy-variables (sparse k-means variant) setting, so method choice should follow the noise structure of the layers.
- In simulation studies KLIC's advantage was demonstrated on synthetic layers with a shared clustering structure and equal/disparate separability; behaviour on real pan-cancer layers should be interpreted via the learned weights rather than assumed.

## Evidence

- [abstract] KLIC frames the challenge of combining clustering structures as a multiple kernel learning problem, in which: "KLIC frames the challenge of combining clustering structures as a multiple kernel learning problem, in which"
- [abstract] noisy datasets down-weighted: "This allows the contribution of noisy datasets to be down-weighted relative to more informative datasets"
- [methods] kernel construction per layer: "For each layer, one kernel is built using Consensus Clustering (Monti et al. 2003) and the most appropriate clustering algorithm."
- [readme] KLIC step of the pipeline: "Integrative clustering is performed using Kernel Learning Integrative Clustering (Cabassi and Kirk, 2020) and the kernels built in the previous step."
