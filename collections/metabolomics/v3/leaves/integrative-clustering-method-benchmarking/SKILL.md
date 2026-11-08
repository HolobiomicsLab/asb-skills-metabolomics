---
name: integrative-clustering-method-benchmarking
description: Use when comparing integrative clustering algorithms across multiple
  omic data layers, especially when some layers are noisy or have different cluster
  separability, and you need ARI-based benchmark tables.
when_to_use_negative:
- When there is only a single data layer to cluster — the skill targets integration
  of multiple omic datasets
- When true labels are unavailable and ARI evaluation is not possible, except for
  the fixed-sigma RBF variant where tuning on true labels is not permitted
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3577
- http://edamontology.org/topic_3366
tools:
- name: klic
  role: Performs Kernel Learning Integrative Clustering on per-layer consensus clustering
    kernels
  repo: https://github.com/acabassi/klic-pancancer-analysis
- name: coca
  role: 'Performs Cluster-Of-Clusters Analysis: k-means per layer followed by hierarchical
    clustering with average linkage'
  repo: https://github.com/acabassi/klic-pancancer-analysis
- name: R
  role: Execution environment; klic and coca are R packages available on CRAN
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btaa593
    title: Multiple kernel learning for integrative consensus clustering of omic datasets
  source_content_hash: sha256:5f8945ded14ded5c582ec3655f46824efbe7cdda34719d11d0339f1b8b2e8cb6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/acabassi/klic-pancancer-analysis
  tools:
  - R
  - coca
  - klic
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3577
  - http://edamontology.org/topic_3366
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# integrative-clustering-method-benchmarking

## Summary

Benchmarks integrative clustering methods (KLIC, COCA, iCluster, Clusternomics, localized multiple kernel k-means with RBF kernels) on simulated multi-omic datasets with varying noise, scoring each by adjusted Rand index against true labels.

## When to use

Use when comparing integrative clustering algorithms across multiple omic data layers, especially when some layers are noisy or have different cluster separability, and you need ARI-based benchmark tables.

## When NOT to use

- When there is only a single data layer to cluster — the skill targets integration of multiple omic datasets
- When true labels are unavailable and ARI evaluation is not possible, except for the fixed-sigma RBF variant where tuning on true labels is not permitted

## Inputs

- Simulated multi-dataset omic data with known true cluster labels (similar-datasets, noisy-variables, noise-levels settings)
- Preprocessed real pan-cancer data layers from Hoadley et al. (2014) downloaded from Synapse (syn2468297)
- Cluster label assignments per layer for COCA input
- Per-layer consensus clustering kernels (SimilarityMatrix objects) for KLIC input

## Outputs

- CSV table of median adjusted Rand index per clustering method per simulation setting
- Cluster assignments per method (COCA clusters, KLIC clusters, iCluster, Clusternomics, RBF-MKL)
- Benchmark ranking comparison against Figure 4 of Cabassi and Kirk (2020)

## How to apply

Generate simulated multi-dataset omic data covering three settings: similar datasets, noisy variables (sparse k-means variant), and noise-levels. Run COCA via the coca package using k-means followed by hierarchical clustering with average linkage (Section 4.1.2 configuration); run iCluster via tune.iCluster2 and Clusternomics via contextCluster. For KLIC, build one kernel per layer using Consensus Clustering (Monti et al. 2003) with the most appropriate clustering algorithm, then perform Kernel Learning Integrative Clustering on the kernels; also run localized multiple kernel k-means with RBF kernels, one with sigma fixed to 1 and one tuned on the true labels. Compute the adjusted Rand index of each method's clustering against the true simulated labels for every setting and aggregate into a CSV of median ARI per method per setting. Verify rankings against Figure 4: RBF best on similar datasets, COCA best with noisy variables, KLIC and tuned RBF best with noisy datasets.

## Related tools

- **klic** (Performs Kernel Learning Integrative Clustering on per-layer consensus clustering kernels) — https://github.com/acabassi/klic-pancancer-analysis
- **coca** (Performs Cluster-Of-Clusters Analysis: k-means per layer followed by hierarchical clustering with average linkage) — https://github.com/acabassi/klic-pancancer-analysis
- **R** (Execution environment; klic and coca are R packages available on CRAN)

## Examples

```
Rscript run_simulations.R  # generate simulated layers, run COCA (coca::coca), iCluster (tune.iCluster2), Clusternomics (contextCluster), KLIC (klic::spectrumShift + klic), compute ARI per setting
```

## Evaluation signals

- ARI is computed against true simulated labels for every method and every setting; median ARI per method per setting is aggregated
- Rankings must match Figure 4: RBF-MKL best on similar datasets, COCA best with noisy variables, KLIC and tuned RBF best with noisy datasets
- KLIC and tuned localized multiple kernel k-means with RBF kernel should achieve the highest ARI when combining datasets with different levels of noise, outperforming unweighted methods
- Kernels for KLIC must be one per layer built via Consensus Clustering (Monti et al. 2003)
- Tuning iCluster via tune.iCluster2 and Clusternomics via contextCluster must complete per simulation setting before scoring

## Limitations

- The contribution of noisy datasets to COCA is unclear; its properties had not been systematically explored before this study
- The sigma-tuned RBF-MKL variant uses true labels, so it serves only as an optimistic upper bound rather than a fully fair comparison
- Real-data application relies on Hoadley et al. (2014) preprocessing; each layer must be preprocessed and clustered in the same way as that study
- No changelog found in the repository, so code version drift may affect reproducibility

## Evidence

- [abstract] R packages klic and coca are available on the Comprehensive R Archive Network: "R packages klic and coca are available on the Comprehensive R Archive Network"
- [abstract] This allows the contribution of noisy datasets to be down-weighted relative to more informative datasets: "This allows the contribution of noisy datasets to be down-weighted relative to more informative datasets"
- [methods] For each layer, one kernel is built using Consensus Clustering (Monti et al. 2003) and the most appropriate clustering algorithm.: "For each layer, one kernel is built using Consensus Clustering (Monti et al. 2003) and the most appropriate clustering algorithm."
- [methods] Each layer of the dataset is preprocessed in the same way as in Hoadley et al. (2014): "Each layer of the  dataset is preprocessed in the same way as in Hoadley et al. (2014)"
- [readme] Integrative clustering is performed using Cluster-Of-Clusters Analysis (Cabassi and Kirk, 2020) and the clusters found in the previous step.: "Integrative clustering is performed using Cluster-Of-Clusters Analysis (Cabassi and Kirk, 2020) and the clusters found in the previous step."
- [readme] The data used for the analysis are those of Hoadley et al. (2014), which can be downloaded from Synapse: "The data used for the analysis are those of Hoadley et al. (2014), which can be downloaded from [Synapse](https://www.synapse.org/#!Synapse:syn2468297/wiki/64259)"
