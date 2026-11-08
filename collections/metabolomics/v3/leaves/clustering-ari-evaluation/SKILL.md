---
name: clustering-ari-evaluation
description: Use when comparing integrative or single-dataset clustering results against
  simulated or known ground-truth labels, e.g. judging whether KLIC's kernel-weighted
  clustering outperforms COCA, iCluster, or single consensus matrices by ARI.
when_to_use_negative:
- Clustering real data with no ground-truth labels — ARI requires true labels, so
  use internal stability or biological validation instead
- Comparing clusterings with different numbers of clusters against each other rather
  than against a reference labeling, where raw ARI interpretation is misleading
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_2269
- http://edamontology.org/topic_3473
tools:
- name: klic
  role: Produces the KLIC integrative clusterings and learned kernel weights whose
    ARI against true labels is computed; available on CRAN
- name: coca
  role: Produces the COCA cluster-of-clusters baseline clusterings that are scored
    against true labels for comparison
- name: R
  role: Environment in which simulations, clustering, and ARI computation/aggregation
    are implemented
provenance:
  source_task_ids:
  - task_002
  - task_003
  - task_005
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btaa593
    title: Multiple kernel learning for integrative consensus clustering of omic datasets
  source_content_hash: sha256:adb432c557f6e3b792f09dc0124e8abc1ac18f24ab81b6c19cbd6849bc428107
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - coca
  - klic
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_2269
  - http://edamontology.org/topic_3473
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# clustering-ari-evaluation

## Summary

Evaluates clustering quality by computing the adjusted Rand index (ARI) between an inferred clustering (e.g. from KLIC, COCA, or kernel k-means) and known true labels, then aggregating ARI (e.g. median or mean over 100 simulation repetitions) to benchmark methods across simulation settings.

## When to use

Use when comparing integrative or single-dataset clustering results against simulated or known ground-truth labels, e.g. judging whether KLIC's kernel-weighted clustering outperforms COCA, iCluster, or single consensus matrices by ARI.

## When NOT to use

- Clustering real data with no ground-truth labels — ARI requires true labels, so use internal stability or biological validation instead
- Comparing clusterings with different numbers of clusters against each other rather than against a reference labeling, where raw ARI interpretation is misleading

## Inputs

- Vector of inferred cluster assignments from a clustering run (e.g. KLIC or kernel k-means output with K=6)
- Vector of true simulated cluster labels
- Per-repetition and per-setting run identifiers (simulation setting, repetition index, method)

## Outputs

- ARI value per clustering result (per setting, method, and repetition)
- Aggregated CSV table of mean or median ARI per method per setting (e.g. simulation_ari_kernel_weights.csv)

## How to apply

After running a clustering method (kernel k-means on a single consensus matrix, KLIC on combined kernels, COCA, iCluster via tune.iCluster2, or Clusternomics via contextCluster), compute the adjusted Rand index between the inferred cluster assignments and the true simulated labels. Repeat this per setting and per repetition (the simulations use 100 repetitions) so results are not driven by a single random initialization. Aggregate per-method ARIs within each setting (median for the five-method benchmark table; mean over repetitions for the noise-levels and similar-datasets simulations) and write them to a CSV alongside per-dataset kernel weights where relevant. Use the aggregated ARIs to check the expected qualitative patterns: KLIC improves ARI over individual consensus matrices on similar datasets, and KLIC and tuned localized multiple kernel k-means (RBF) achieve the highest ARI when combining noisy datasets.

## Related tools

- **klic** (Produces the KLIC integrative clusterings and learned kernel weights whose ARI against true labels is computed; available on CRAN)
- **coca** (Produces the COCA cluster-of-clusters baseline clusterings that are scored against true labels for comparison)
- **R** (Environment in which simulations, clustering, and ARI computation/aggregation are implemented)

## Evaluation signals

- ARI must be bounded in [-1, 1]; near-1 indicates recovery of the 6 equal simulated clusters
- In the similar-datasets simulation, ARI for the four-dataset KLIC combination (A+B+C+D) must exceed the ARI of kernel k-means on each single consensus matrix
- In the noise-levels simulation, ARI must increase as more informative (higher-separability) layers are included in the three-way KLIC subsets
- Median-ARI rankings in the benchmark table must match the expected pattern (RBF best on similar datasets; COCA best with noisy variables; KLIC and tuned RBF best with noisy datasets)
- Aggregated CSV contains one median/mean ARI per method per setting over all 100 repetitions, not a single unreplicated value

## Limitations

- ARI evaluation is only meaningful in simulation settings where true labels are known; real-data applications (e.g. pan-cancer subtyping) cannot be scored this way
- ARI is sensitive to the number of clusters fixed for the method (e.g. K=6 in the simulations) and to random initialization, hence the need for 100 repetitions
- Median-ARI comparisons depend on the simulation settings used (similar datasets, noisy variables, noise levels) and may not generalize to other data structures

## Evidence

- [abstract] We compare the performances of KLIC and COCA in a variety of situations through simulation studies.: "We compare the performances of KLIC and COCA in a variety of situations through simulation studies."
- [other] compute the adjusted Rand index (ARI) between the KLIC clustering and the true labels: [evidence span withheld — anchor status: not_found]
- [results] Combine the four similar datasets with KLIC: [evidence span withheld — anchor status: not_found]
- [readme] The code contained in this folder can be used to reproduce the pancancer analysis presented in Cabassi and Kirk (2020).: "The code contained in this folder can be used to reproduce the pancancer analysis presented in Cabassi and Kirk (2020)."
