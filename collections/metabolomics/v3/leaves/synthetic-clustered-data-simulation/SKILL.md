---
name: synthetic-clustered-data-simulation
description: Use when evaluating or validating an integrative clustering method (e.g.
  KLIC, COCA, kernel k-means) on simulated datasets where the true labels and per-dataset
  noise levels are known and kernel-weight allocation must be checked.
when_to_use_negative:
- Real omic data where true cluster labels are unknown — ARI against ground truth
  cannot be computed and the noise-level validation pattern is unobservable
- Datasets whose layers do not share the same clustering structure over the same samples
  — the weighted-down-noisy-layer behaviour being tested presumes a common underlying
  partition
- Reproducing the pan-cancer analysis on Hoadley et al. (2014) data — use the repository's
  Preprocessing/COCA/Kernel/KLIC workflow instead of the synthetic simulation
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: klic
  role: CRAN package providing Consensus Clustering for kernel construction and KLIC
    (multiple kernel k-means with kernel weight learning) for the simulated subsets
  repo: https://www.synapse.org/#!Synapse:syn2468297/wiki/64259
- name: coca
  role: Companion CRAN package (COCA) used as the comparison integrative clustering
    method in the simulation studies
- name: R
  role: Language in which the synthetic datasets are generated and the whole simulation
    (Section 3.1 parameters) is implemented
provenance:
  source_task_ids:
  - task_002
  - task_004
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btaa593
    title: Multiple kernel learning for integrative consensus clustering of omic datasets
  source_content_hash: sha256:5b7ed34609ba69f269ca974109774bfd4f80d231040bb9e17b8d3f6c31bbc864
schema_version: 0.2.0
metadata:
  license_tier: open
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
yes_rate: 0.9230769230769231
n_claims: 13
---

# synthetic-clustered-data-simulation

## Summary

Generates synthetic multi-layer datasets with controlled clustering structure (varying separability/noise levels 0–3) to benchmark integrative clustering methods such as KLIC, verifying that kernel weights are allocated to the clearest (highest-separability) dataset and that ARI behaves as expected.

## When to use

Use when evaluating or validating an integrative clustering method (e.g. KLIC, COCA, kernel k-means) on simulated datasets where the true labels and per-dataset noise levels are known and kernel-weight allocation must be checked.

## When NOT to use

- Real omic data where true cluster labels are unknown — ARI against ground truth cannot be computed and the noise-level validation pattern is unobservable
- Datasets whose layers do not share the same clustering structure over the same samples — the weighted-down-noisy-layer behaviour being tested presumes a common underlying partition
- Reproducing the pan-cancer analysis on Hoadley et al. (2014) data — use the repository's Preprocessing/COCA/Kernel/KLIC workflow instead of the synthetic simulation

## Inputs

- Synthetic simulation parameters (methods Section 3.1: four datasets with separability levels 0, 1, 2, 3)
- Known true cluster labels per synthetic dataset
- Per-layer consensus (similarity) kernel matrices built with Consensus Clustering via the klic package

## Outputs

- simulation_ari_kernel_weights.csv with averaged ARIs and per-dataset kernel weights over 100 repetitions
- Per-subset KLIC cluster assignments (true-label ARI per repetition)
- Per-dataset average kernel weights from the multiple kernel learning step

## How to apply

Generate synthetic datasets with the simulation parameters of methods Section 3.1, spanning separability levels 0–3, implemented in R. For each dataset, build a similarity kernel via Consensus Clustering (Monti et al. 2003) as implemented in the CRAN klic package, choosing the most appropriate base clustering algorithm for each layer. Run kernel k-means per dataset as a single-dataset baseline, then run KLIC on each three-way subset of layers (0+1+2, 0+1+3, 0+2+3, 1+2+3) over 100 simulation repetitions. For each repetition, compute the adjusted Rand index (ARI) against the true labels and record the per-dataset kernel weights from the multiple kernel learning step; average over repetitions (e.g. to simulation_ari_kernel_weights.csv). The rationale is that KLIC frames integrative clustering as a multiple kernel learning problem, so a correctly functioning implementation should down-weight the noisy (separability-level 0) layer relative to the clearest one.

## Related tools

- **klic** (CRAN package providing Consensus Clustering for kernel construction and KLIC (multiple kernel k-means with kernel weight learning) for the simulated subsets) — https://www.synapse.org/#!Synapse:syn2468297/wiki/64259
- **coca** (Companion CRAN package (COCA) used as the comparison integrative clustering method in the simulation studies)
- **R** (Language in which the synthetic datasets are generated and the whole simulation (Section 3.1 parameters) is implemented)

## Examples

```
install.packages('klic'); library(klic)  # then build per-layer consensus kernels and run KLIC on each three-way separability subset (0+1+2, 0+1+3, 0+2+3, 1+2+3)
```

## Evaluation signals

- In each three-way subset, the dataset with the highest separability level receives the largest average kernel weight, and the separability-level-0 (noisiest) layer receives the smallest
- Average ARI against true labels increases as more informative (higher-separability) layers are included in the subset
- Per-dataset kernel weights are valid non-negative weights summing to 1 over the kernels in each subset
- Results are stable across the 100 simulation repetitions (qualitative reproduction of Fig. 3 pattern)
- Single-dataset kernel k-means baselines complete for all four separability levels, giving a reference ARI per layer

## Limitations

- The finding is qualitative: KLIC assigns higher average weights to the clearest dataset, but the simulation does not establish a quantitative threshold on kernel weights
- The synthetic design assumes all layers share the same clustering structure and differ only in noise/separability, so conclusions do not cover layers with conflicting structures
- Simulation conclusions rely on 100 repetitions and Consensus Clustering with a per-layer choice of base clustering algorithm, which introduces algorithm-selection sensitivity

## Evidence

- [abstract] This allows the contribution of noisy datasets to be down-weighted relative to more informative datasets: "This allows the contribution of noisy datasets to be down-weighted relative to more informative datasets"
- [abstract] We compare the performances of KLIC and COCA in a variety of situations through simulation studies.: "We compare the performances of KLIC and COCA in a variety of situations through simulation studies."
- [readme] For each layer, one kernel is built using Consensus Clustering (Monti et al. 2003) and the most appropriate clustering algorithm.: "For each layer, one kernel is built using Consensus Clustering (Monti et al. 2003) and the most appropriate clustering algorithm."
- [readme] Integrative clustering is performed using Kernel Learning Integrative Clustering (Cabassi and Kirk, 2020) and the kernels built in the previous step.: "Integrative clustering is performed using Kernel Learning Integrative Clustering (Cabassi and Kirk, 2020) and the kernels built in the previous step."
