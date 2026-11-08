---
name: simulated-multi-omics-dataset-generation
description: Use when benchmarking integrative clustering methods on multi-dataset
  omic data and you need ground-truth labels across settings like similar datasets,
  noisy variables (sparse k-means variant), and datasets with different noise levels.
when_to_use_negative:
- The analysis target is real pan-cancer data (Hoadley et al. 2014 from Synapse syn2468297),
  where no ground-truth labels exist and ARI against true labels cannot be computed
- Only a single data layer is available, since the skill is designed for combining
  datasets with different levels of noise, not single-view clustering
tools:
- name: klic
  role: R package used to run Kernel Learning Integrative Clustering on consensus-clustering
    kernels built per simulated data layer
  repo: https://github.com/acabassi/klic-pancancer-analysis
- name: coca
  role: R package providing Cluster-Of-Clusters Analysis (k-means then hierarchical
    clustering with average linkage) as a comparison method on simulated settings
  repo: https://github.com/acabassi/klic-pancancer-analysis
- name: R
  role: Language in which simulation scripts, preprocessing, and all five clustering
    benchmarks are executed
  repo: https://www.synapse.org/#!Synapse:syn2468297/wiki/64259
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btaa593
    title: Multiple kernel learning for integrative consensus clustering of omic datasets
  source_content_hash: sha256:4c84c27f60dd8caf4e5243339854a7f14d7b2ec24d4c4bff519cb99800415039
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/acabassi/klic-pancancer-analysis
  tools:
  - R
  - coca
  - klic
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 14
---

# simulated-multi-omics-dataset-generation

## Summary

Generates simulated multi-dataset (multi-omic) data with controlled cluster separability and noise levels to benchmark integrative clustering methods such as KLIC, COCA, iCluster, and Clusternomics via median adjusted Rand index.

## When to use

Use when benchmarking integrative clustering methods on multi-dataset omic data and you need ground-truth labels across settings like similar datasets, noisy variables (sparse k-means variant), and datasets with different noise levels.

## When NOT to use

- The analysis target is real pan-cancer data (Hoadley et al. 2014 from Synapse syn2468297), where no ground-truth labels exist and ARI against true labels cannot be computed
- Only a single data layer is available, since the skill is designed for combining datasets with different levels of noise, not single-view clustering

## Inputs

- Simulation parameters defining cluster separability, number of noisy variables, and per-dataset noise levels
- Simulation scripts from the klic-pancancer-analysis R repository
- True simulated cluster labels per setting

## Outputs

- Simulated multi-dataset omic matrices per setting (similar-datasets, noisy-variables, noise-levels)
- Per-method cluster assignments for each simulation setting
- CSV table of median adjusted Rand index per method per setting (KLIC, COCA, iCluster, Clusternomics, RBF-MKL)

## How to apply

Generate simulated multi-dataset omic data using the simulation scripts and parameters from the klic-pancancer-analysis repository, covering the similar-datasets, noisy-variables (sparse k-means variant), and noise-levels settings where datasets differ in cluster separability. Retain the true simulated labels so that each method's clustering can be scored with the adjusted Rand index. For each layer, build one kernel using Consensus Clustering (Monti et al. 2003) with the most appropriate clustering algorithm, then run KLIC from the klic package; additionally run COCA (k-means followed by hierarchical clustering with average linkage, per Section 4.1.2), iCluster via tune.iCluster2, Clusternomics via contextCluster, and localized multiple kernel k-means with RBF kernels (sigma fixed to 1 and a variant tuned on the true labels). The rationale is that KLIC's multiple kernel learning weights allow the contribution of noisy datasets to be down-weighted relative to more informative datasets, which the simulation settings are designed to probe. Aggregate per-setting ARI values into a median-ARI table per method per setting to reproduce the five-method benchmark rankings.

## Related tools

- **klic** (R package used to run Kernel Learning Integrative Clustering on consensus-clustering kernels built per simulated data layer) — https://github.com/acabassi/klic-pancancer-analysis
- **coca** (R package providing Cluster-Of-Clusters Analysis (k-means then hierarchical clustering with average linkage) as a comparison method on simulated settings) — https://github.com/acabassi/klic-pancancer-analysis
- **R** (Language in which simulation scripts, preprocessing, and all five clustering benchmarks are executed) — https://www.synapse.org/#!Synapse:syn2468297/wiki/64259

## Evaluation signals

- Median ARI rankings across settings match Figure 4: RBF-MKL best on similar datasets; COCA best with noisy variables; KLIC and tuned-RBF best with noisy datasets
- KLIC and tuned localized multiple kernel k-means with RBF kernel achieve the highest ARI scores, outperforming the unweighted methods in noisy-dataset settings
- Every method's clustering is scored against true simulated labels with the adjusted Rand index for every setting (no missing cells in the CSV table)
- Each simulation setting (similar-datasets, noisy-variables sparse k-means variant, noise-levels) is represented in the aggregated results

## Limitations

- The article notes the properties of COCA had never been systematically explored and its robustness to the inclusion of noisy datasets is unclear, so benchmark outcomes depend on the specific noise configurations chosen
- RBF-MKL has a tuned variant that uses the true labels for sigma tuning, which is not a realistic supervised setting and may inflate its ARI relative to fully unsupervised KLIC
- Repository README provides no changelog, so script behavior may drift from the published 2020 analysis

## Evidence

- [abstract] noise weighting rationale: "This allows the contribution of noisy datasets to be down-weighted relative to more informative datasets"
- [abstract] benchmark comparison scope: "We compare the performances of KLIC and COCA in a variety of situations through simulation studies."
- [methods] kernel construction step: "For each layer, one kernel is built using Consensus Clustering (Monti et al. 2003) and the most appropriate clustering algorithm."
- [readme] reproducibility of pan-cancer analysis: "The code contained in this folder can be used to reproduce the pancancer analysis presented in Cabassi and Kirk (2020)."
