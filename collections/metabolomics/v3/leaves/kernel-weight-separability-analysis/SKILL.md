---
name: kernel-weight-separability-analysis
description: 'Use when you need to demonstrate or validate that a multiple kernel
  learning integrative clustering method down-weights noisy datasets: i.e., you have
  kernels from datasets with identical clustering structure but graded cluster separability
  and want per-dataset kernel weights.'
when_to_use_negative:
- The datasets do not share the same underlying clustering structure — then kernel
  weights reflect genuine structural differences, not robustness to noise
- You need the real-data pan-cancer (Hoadley et al. 2014) analysis rather than the
  simulation study — use the klic-pancancer-analysis repo pipeline (Preprocessing,
  COCA, Kernel, KLIC) instead
- You want COCA (Cluster-of-Clusters Analysis) behaviour; this skill probes KLIC's
  multiple kernel learning weights specifically
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3307
- http://edamontology.org/topic_0092
tools:
- name: klic
  role: CRAN R package performing Kernel Learning Integrative Clustering and the multiple
    kernel learning step that returns per-dataset kernel weights
  repo: https://www.synapse.org/#!Synapse:syn2468297/wiki/64259
- name: coca
  role: CRAN R package for Cluster-Of-Clusters Analysis, the comparator integrative
    clustering method in the simulation study
- name: R
  role: Language used to generate the synthetic separability-level datasets, run the
    100 simulation repetitions, and compute averaged ARIs and kernel weights
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btaa593
    title: Multiple kernel learning for integrative consensus clustering of omic datasets
  source_content_hash: sha256:43a3aabf4b4eae4207f47a17afd7961d02b819b709d5f19a2fddc3fefe48ce59
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
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# kernel-weight-separability-analysis

## Summary

A simulation-based diagnostic for KLIC (Kernel Learning Integrative Clustering, Cabassi and Kirk 2020) that verifies its multiple kernel learning step assigns larger weights to datasets with clearer clustering structure, by combining synthetic datasets that share a structure but differ in cluster separability (noise levels 0-3).

## When to use

Use when you need to demonstrate or validate that a multiple kernel learning integrative clustering method down-weights noisy datasets: i.e., you have kernels from datasets with identical clustering structure but graded cluster separability and want per-dataset kernel weights.

## When NOT to use

- The datasets do not share the same underlying clustering structure — then kernel weights reflect genuine structural differences, not robustness to noise
- You need the real-data pan-cancer (Hoadley et al. 2014) analysis rather than the simulation study — use the klic-pancancer-analysis repo pipeline (Preprocessing, COCA, Kernel, KLIC) instead
- You want COCA (Cluster-of-Clusters Analysis) behaviour; this skill probes KLIC's multiple kernel learning weights specifically

## Inputs

- Four synthetic datasets sharing the same clustering structure but differing in noise/cluster separability (levels 0, 1, 2, 3), generated per methods Section 3.1 parameters
- Per-dataset similarity kernels built with Consensus Clustering (Monti et al. 2003) using the most appropriate base clustering algorithm
- True cluster labels for ARI computation

## Outputs

- simulation_ari_kernel_weights.csv: averaged ARIs and per-dataset kernel weights per three-way subset over 100 repetitions
- Qualitative reproduction of Fig. 3: kernel weight ordering across subsets (0,1,2), (0,1,3), (0,2,3), (1,2,3)

## How to apply

Generate the four synthetic datasets with separability levels 0, 1, 2 and 3 using the simulation parameters of methods Section 3.1, implemented in R. For each dataset, build a similarity kernel with Consensus Clustering (Monti et al. 2003), choosing the most appropriate base clustering algorithm as in the Kernel step of the pipeline. Run kernel k-means on each dataset individually over 100 simulation repetitions as a single-dataset baseline. Run KLIC from the CRAN klic package on each three-way subset (0+1+2, 0+1+3, 0+2+3, 1+2+3) using those kernels, over 100 repetitions, computing the adjusted Rand index (ARI) against true labels and recording the kernel weight assigned to each dataset by the multiple kernel learning step. Average ARIs and per-dataset kernel weights over the 100 repetitions and write them to simulation_ari_kernel_weights.csv; the expected qualitative pattern (Fig. 3) is that the separability-0 layer receives the smallest average kernel weight in every three-way subset while ARI increases with the number of informative layers included.

## Related tools

- **klic** (CRAN R package performing Kernel Learning Integrative Clustering and the multiple kernel learning step that returns per-dataset kernel weights) — https://www.synapse.org/#!Synapse:syn2468297/wiki/64259
- **coca** (CRAN R package for Cluster-Of-Clusters Analysis, the comparator integrative clustering method in the simulation study)
- **R** (Language used to generate the synthetic separability-level datasets, run the 100 simulation repetitions, and compute averaged ARIs and kernel weights)

## Examples

```
library(klic); klic_results <- klic(list(sim0_kernel, sim1_kernel, sim2_kernel), M = 3)  # three-way subset, weights in klic_results$weights
```

## Evaluation signals

- In each three-way subset, the dataset with separability level 0 (the noisy layer) receives the smallest average kernel weight
- The clearest (highest-separability) dataset receives higher average kernel weight than the other two datasets in every three-way combination
- ARI between KLIC clustering and true labels increases as more informative layers are included in the subset
- Averages are stable over the 100 simulation repetitions (per-repetition weights and ARIs recorded before averaging)
- Results qualitatively reproduce Fig. 3 of Cabassi and Kirk (2020)

## Limitations

- The result is a qualitative pattern, not an exact numeric match; validation is by qualitative reproduction of Fig. 3
- The synthetic datasets deliberately share the same clustering structure, so conclusions apply to noise/separability robustness only
- Kernel construction depends on selecting 'the most appropriate clustering algorithm' for each layer via Consensus Clustering, a modelling decision that can affect weights

## Evidence

- [abstract] This allows the contribution of noisy datasets to be down-weighted relative to more informative datasets: "This allows the contribution of noisy datasets to be down-weighted relative to more informative datasets"
- [abstract] We compare the performances of KLIC and COCA in a variety of situations through simulation studies.: "We compare the performances of KLIC and COCA in a variety of situations through simulation studies."
- [methods] For each layer, one kernel is built using Consensus Clustering (Monti et al. 2003) and the most appropriate clustering algorithm.: "For each layer, one kernel is built using Consensus Clustering (Monti et al. 2003) and the most appropriate clustering algorithm."
- [readme] The code contained in this folder can be used to reproduce the pancancer analysis presented in Cabassi and Kirk (2020).: "The code contained in this folder can be used to reproduce the pancancer analysis presented in Cabassi and Kirk (2020)."
- [other] Run KLIC (Kernel Learning Integrative Clustering, Cabassi and Kirk 2020) from the CRAN klic package on each three-way subset of datasets: [evidence span withheld — anchor status: not_found]
