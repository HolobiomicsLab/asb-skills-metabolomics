---
name: multi-omics-integrative-clustering
description: Use when you have independent clusterings (or per-dataset kernels) over
  the same set of observations from M omic platforms and need a single global clustering
  that down-weights noisy datasets, e.g. pan-cancer subtyping or transcriptional module
  discovery.
when_to_use_negative:
- You only have a single omic layer and no per-dataset clusterings or kernels — integrative
  consensus clustering has nothing to combine
- You need raw feature-level integration (e.g. regression or classifier features)
  rather than integration of precomputed cluster structures; KLIC and COCA operate
  on kernels/clusterings, not raw feature tables
- Datasets cover non-overlapping sets of observations, since the MOC is N-column binary
  over a shared observation set and kernels must be defined on the same samples
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: klic
  role: Builds per-dataset consensus kernels (Monti et al. 2003) and runs Kernel Learning
    Integrative Clustering to learn per-dataset kernel weights
  repo: https://www.synapse.org/#!Synapse:syn2468297/wiki/64259
- name: coca
  role: 'Runs Cluster-Of-Clusters Analysis: converts per-dataset clusterings into
    the binary Matrix Of Clusters (MOC) and consensus-clusters it'
  repo: https://github.com/acabassi/klic-pancancer-analysis
- name: R
  role: Analysis environment; the klic and coca packages are distributed on CRAN and
    the reproduction scripts are R scripts
  repo: https://github.com/acabassi/klic-pancancer-analysis
provenance:
  source_task_ids:
  - task_006
  - task_007
  - task_008
  source_papers:
  - doi: 10.1093/bioinformatics/btaa593
    title: Multiple kernel learning for integrative consensus clustering of omic datasets
  source_content_hash: sha256:f2d18afe8572db2784ada6c261b3847d75c7b81063cba33e801303c424e59138
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - coca
  - klic
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# multi-omics-integrative-clustering

## Summary

Integrative clustering of multiple omic datasets via Kernel Learning Integrative Clustering (KLIC), which frames consensus clustering as a multiple kernel learning problem with learned per-dataset weights, and its COCA alternative for combining per-dataset clusterings.

## When to use

Use when you have independent clusterings (or per-dataset kernels) over the same set of observations from M omic platforms and need a single global clustering that down-weights noisy datasets, e.g. pan-cancer subtyping or transcriptional module discovery.

## When NOT to use

- You only have a single omic layer and no per-dataset clusterings or kernels — integrative consensus clustering has nothing to combine
- You need raw feature-level integration (e.g. regression or classifier features) rather than integration of precomputed cluster structures; KLIC and COCA operate on kernels/clusterings, not raw feature tables
- Datasets cover non-overlapping sets of observations, since the MOC is N-column binary over a shared observation set and kernels must be defined on the same samples

## Inputs

- Per-dataset cluster label vectors c_m for M datasets (from preprocessing each layer as in Hoadley et al. 2014)
- One consensus similarity kernel per dataset, built via Consensus Clustering (Monti et al. 2003)
- Multiplatform omic layers, e.g. Hoadley et al. (2014) pan-cancer data from Synapse syn2468297 (DNA copy number, methylation, mRNA, miRNA, protein) or Granovskaia et al. (2010) yeast expression (551 genes x 41 time points) plus Harbison et al. (2004) ChIP-chip (117 regulators, discretised)

## Outputs

- Global integrated clustering of the N observations at the selected K
- Learned per-dataset kernel weights (e.g. klic_kernel_weights.csv: copy number 31.4%, methylation 19.2%, miRNA 17.8%, mRNA 16.4%, protein 15.2%)
- Average silhouette width versus K table (e.g. coca_silhouette_table.csv)
- Matrix Of Clusters (MOC), a K̄×N binary CSV (moc_matrix.csv) encoding cluster membership per dataset (COCA path)
- GOTO score tables per ontology for comparing KLIC vs COCA and single-dataset clusterings

## How to apply

For each layer of the dataset, build one similarity kernel using Consensus Clustering (Monti et al. 2003) with the most appropriate clustering algorithm for that layer (e.g. PAM for yeast expression, BHC for ChIP-chip), implemented in the klic R package. Run KLIC over a candidate range of K (e.g. K=2..20), compute the average silhouette width for each K, and select the K maximising it (e.g. K=10 for the Hoadley et al. 2014 12-cancer-type data, K=4 for the yeast expression/ChIP-chip integration). Alternatively, run COCA: stack per-dataset cluster label vectors into a binary Matrix Of Clusters (MOC) of size K̄×N, where K̄ is the sum of per-dataset cluster counts, then consensus-cluster the MOC. Compare approaches with GOTO (Gene Ontology Term Overlap) scores per ontology (biological process, molecular function, cellular component) and inspect learned kernel weights, since KLIC's weights allow the contribution of noisy datasets (e.g. the noisier ChIP-chip kernel) to be down-weighted relative to more informative ones.

## Related tools

- **klic** (Builds per-dataset consensus kernels (Monti et al. 2003) and runs Kernel Learning Integrative Clustering to learn per-dataset kernel weights) — https://www.synapse.org/#!Synapse:syn2468297/wiki/64259
- **coca** (Runs Cluster-Of-Clusters Analysis: converts per-dataset clusterings into the binary Matrix Of Clusters (MOC) and consensus-clusters it) — https://github.com/acabassi/klic-pancancer-analysis
- **R** (Analysis environment; the klic and coca packages are distributed on CRAN and the reproduction scripts are R scripts) — https://github.com/acabassi/klic-pancancer-analysis

## Examples

```
install.packages(c('klic','coca'))  # R packages klic and coca are available on the Comprehensive R Archive Network
```

## Evaluation signals

- Silhouette curve over candidate K attains its maximum at the expected K (K=10 for the Hoadley pan-cancer data; K=4 for the yeast KLIC solution)
- Learned kernel weights match reported values within rounding tolerance: copy number 31.4%, methylation 19.2%, miRNA 17.8%, mRNA 16.4%, protein 15.2%
- MOC validation: CSV has exactly K̄ rows and N columns, contains only binary (0/1) entries, and matches the hand-computed reference matrix for a synthetic example
- GOTO score table shows KLIC down-weighting the noisier ChIP-chip kernel relative to the expression kernel, reproducing Table 1 of Cabassi and Kirk (2020)
- COCA run on the yeast data selects K=10, consistent with the paper's reported COCA solution

## Limitations

- COCA's properties had never been systematically explored and its robustness to the inclusion of noisy datasets is unclear — KLIC is preferred when down-weighting noisy datasets matters
- Performance comparisons rely on simulation studies and the two real applications; behaviour outside these settings is unverified
- Results depend on the per-layer clustering algorithm chosen for Consensus Clustering ('the most appropriate clustering algorithm'), e.g. PAM vs BHC, which is a manual decision point

## Evidence

- [abstract] KLIC frames the challenge of combining clustering structures as a multiple kernel learning problem: "present Kernel Learning Integrative Clustering (KLIC) as an alternative strategy. KLIC frames the challenge of combining clustering structures as a multiple kernel learning problem, in which"
- [abstract] Down-weighting of noisy datasets: "This allows the contribution of noisy datasets to be down-weighted relative to more informative datasets"
- [methods] Kernel construction per layer: "For each layer, one kernel is built using Consensus Clustering (Monti et al. 2003) and the most appropriate clustering algorithm."
- [abstract] Tool availability on CRAN: "R packages klic and coca are available on the Comprehensive R Archive Network"
- [readme] Reproducible pipeline structure: "1) **COCA**. Integrative clustering is performed using Cluster-Of-Clusters Analysis (Cabassi and Kirk, 2020) and the clusters found in the previous step."
