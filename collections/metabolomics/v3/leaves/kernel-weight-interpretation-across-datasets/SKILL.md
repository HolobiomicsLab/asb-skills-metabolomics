---
name: kernel-weight-interpretation-across-datasets
description: Use when an integrative clustering analysis combines multiple consensus-matrix
  kernels (e.g., four omic layers or simulated datasets A–D) with KLIC and you need
  to read the learned kernel weights as evidence of each dataset's relative information
  content, alongside ARI-based cluster recovery.
when_to_use_negative:
- Only a single dataset/kernel is available — kernel weights require combining multiple
  datasets, and there is nothing to distribute weight across
- The integrative method is COCA (Cluster-Of-Clusters Analysis), which does not learn
  per-dataset weights, so no weight interpretation is possible
- Input is raw feature data rather than consensus-matrix kernels — kernels must first
  be built via Consensus Clustering before weights exist
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_3473
- http://edamontology.org/topic_3366
tools:
- name: klic
  role: Builds consensus-matrix kernels via Consensus Clustering and runs KLIC, returning
    the per-dataset kernel weights that this skill interprets
  repo: github.com/acabassi/klic-pancancer-analysis
- name: coca
  role: Companion R package; COCA serves as the non-weighted alternative against which
    KLIC's weighted multiple kernel learning framing is contrasted
- name: R
  role: Implementation language for the simulation repetitions, ARI computation, and
    CSV aggregation of weights and ARI values
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btaa593
    title: Multiple kernel learning for integrative consensus clustering of omic datasets
  source_content_hash: sha256:970c8a8dd4d4cd5ab9ca147e305179a7ca23b4a5f1609244f6069629fc6dc9ff
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - coca
  - klic
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_3473
  - http://edamontology.org/topic_3366
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# kernel-weight-interpretation-across-datasets

## Summary

Interprets the per-dataset kernel coefficients learned by KLIC (Kernel Learning Integrative Clustering) as a diagnostic of each dataset's informativeness and dispersion: equal, similarly dispersed datasets receive roughly equal weights, while noisy or less informative layers are down-weighted.

## When to use

Use when an integrative clustering analysis combines multiple consensus-matrix kernels (e.g., four omic layers or simulated datasets A–D) with KLIC and you need to read the learned kernel weights as evidence of each dataset's relative information content, alongside ARI-based cluster recovery.

## When NOT to use

- Only a single dataset/kernel is available — kernel weights require combining multiple datasets, and there is nothing to distribute weight across
- The integrative method is COCA (Cluster-Of-Clusters Analysis), which does not learn per-dataset weights, so no weight interpretation is possible
- Input is raw feature data rather than consensus-matrix kernels — kernels must first be built via Consensus Clustering before weights exist

## Inputs

- One consensus-matrix kernel per dataset/omic layer (built via Consensus Clustering with the klic package)
- Number of clusters K (e.g., K=6 for the six-equal-cluster simulation)
- True cluster labels for computing ARI in simulation settings (e.g., 100 repetitions of four bivariate-normal datasets A–D with separation parameter s)

## Outputs

- Learned kernel weight per dataset from KLIC (averaged over repetitions)
- ARI values per setting (single-dataset A, B, C, D vs combined A+B+C+D)
- CSV table of ARI and average kernel weights reproducing a Fig. 2-style comparison

## How to apply

First build one kernel per dataset/layer using Consensus Clustering (Monti et al. 2003) as implemented in the klic R package, choosing the most appropriate clustering algorithm per layer. Run KLIC (Cabassi and Kirk 2020) on the combined kernels with the chosen number of clusters (e.g., K=6 for the six-equal-clusters simulation with 300 observations), which frames integrative clustering as a multiple kernel learning problem and returns a learned weight per dataset. To interpret the weights, repeat the analysis over many repetitions (e.g., 100 simulated replicates) and average the kernel weights per dataset, then compare the averaged weights across datasets: in the Sec. 3.1 'similar datasets' simulation with equal separation parameter s and identity covariance, the average weights divide equally among A, B, C, D, whereas noisy layers should be down-weighted relative to more informative ones. Validate the interpretation by computing the Adjusted Rand Index against true labels for each single-dataset kernel k-means baseline and for the combined KLIC result, checking that higher-weighted (or equally weighted informative) datasets correspond to better cluster recovery.

## Related tools

- **klic** (Builds consensus-matrix kernels via Consensus Clustering and runs KLIC, returning the per-dataset kernel weights that this skill interprets) — github.com/acabassi/klic-pancancer-analysis
- **coca** (Companion R package; COCA serves as the non-weighted alternative against which KLIC's weighted multiple kernel learning framing is contrasted)
- **R** (Implementation language for the simulation repetitions, ARI computation, and CSV aggregation of weights and ARI values)

## Examples

```
library(klic); klic_fit <- klic(consensus_matrices, K = 6); weights <- klic_fit$weights
```

## Evaluation signals

- In the equal-separability simulation (same s, identity covariance), the average kernel weights across 100 repetitions should be approximately equal among datasets A, B, C, D
- Kernel weights for deliberately noisy datasets should be down-weighted relative to more informative datasets
- The combined A+B+C+D KLIC clustering should achieve a higher ARI against true labels than kernel k-means on any single consensus matrix
- Weight vectors should be reproducible/consistent in distribution across independent repetitions
- A CSV table of per-setting ARI values and average weights should reproduce the article's Fig. 2 pattern

## Limitations

- The equal-weights result is specific to the 'similar datasets' setting; with datasets of unequal dispersion or informativeness the weight distribution will differ and must be re-examined
- Weight interpretation was validated through simulation and the Hoadley et al. (2014) pan-cancer application, not as a general guarantee for arbitrary kernel inputs
- COCA, the alternative approach, provides no per-dataset weighting, so cross-method comparison of weights is not possible

## Evidence

- [abstract] KLIC frames the challenge of combining clustering structures as a multiple kernel learning problem: "KLIC frames the challenge of combining clustering structures as a multiple kernel learning problem, in which"
- [abstract] This allows the contribution of noisy datasets to be down-weighted relative to more informative datasets: "This allows the contribution of noisy datasets to be down-weighted relative to more informative datasets"
- [methods] For each layer, one kernel is built using Consensus Clustering (Monti et al. 2003) and the most appropriate clustering algorithm.: "For each layer, one kernel is built using Consensus Clustering (Monti et al. 2003) and the most appropriate clustering algorithm."
- [methods] Integrative clustering is performed using Kernel Learning Integrative Clustering (Cabassi and Kirk, 2020) and the kernels built in the previous step.: "Integrative clustering is performed using Kernel Learning Integrative Clustering (Cabassi and Kirk, 2020) and the kernels built in the previous step."
- [readme] The code contained in this folder can be used to reproduce the pancancer analysis presented in Cabassi and Kirk (2020).: "The code contained in this folder can be used to reproduce the pancancer analysis presented in Cabassi and Kirk (2020)."
