---
name: simulated-multidataset-clustering-generation
description: Use when benchmarking KLIC/COCA-style integrative consensus clustering
  on synthetic data with known true labels — e.g. to quantify ARI gains of combining
  kernels versus single-dataset clustering and to inspect learned kernel weights.
when_to_use_negative:
- Input is real multi-omic data (e.g. the Hoadley et al. 2014 pan-cancer layers) rather
  than a controlled simulation with known true labels — use the pan-cancer analysis
  workflow instead
- The synthetic datasets are meant to be noisy or unequally informative rather than
  similar; the article's weighting rationale (down-weighting noisy datasets) calls
  for a different simulation setting
- The goal is a single-dataset clustering problem with no multiple data sources to
  integrate — kernel combination adds no information here
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_2269
tools:
- name: klic
  role: Generates the simulated similar datasets, builds consensus-matrix kernels
    via Consensus Clustering, and performs Kernel Learning Integrative Clustering
    with learned kernel weights
  repo: https://www.synapse.org/#!Synapse:syn2468297/wiki/64259
- name: coca
  role: Companion R package from the same authors providing the consensus-clustering
    machinery against which KLIC is benchmarked in the simulation studies
- name: R
  role: Language in which the simulation scripts and 100-repetition benchmark are
    run
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btaa593
    title: Multiple kernel learning for integrative consensus clustering of omic datasets
  source_content_hash: sha256:fdd6fdcf8066c4fe13f30ddf9f32125e686d1134146c0d632c12d09e8bfb7cec
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
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# simulated-multidataset-clustering-generation

## Summary

Generates the 'similar datasets' simulation benchmark from Cabassi & Kirk (2020): 100 repetitions of four synthetic datasets (A–D), each with 300 observations in 6 equal bivariate-normal clusters at the same separability s, used to demonstrate that KLIC over combined consensus kernels yields higher ARI than clustering each dataset alone.

## When to use

Use when benchmarking KLIC/COCA-style integrative consensus clustering on synthetic data with known true labels — e.g. to quantify ARI gains of combining kernels versus single-dataset clustering and to inspect learned kernel weights.

## When NOT to use

- Input is real multi-omic data (e.g. the Hoadley et al. 2014 pan-cancer layers) rather than a controlled simulation with known true labels — use the pan-cancer analysis workflow instead
- The synthetic datasets are meant to be noisy or unequally informative rather than similar; the article's weighting rationale (down-weighting noisy datasets) calls for a different simulation setting
- The goal is a single-dataset clustering problem with no multiple data sources to integrate — kernel combination adds no information here

## Inputs

- Simulation configuration: 100 repetitions, four datasets (A, B, C, D), 300 observations each, 6 equal clusters, bivariate normal distributions with identity covariance matrix and separation parameter s
- True cluster labels for each simulated observation

## Outputs

- Per-dataset consensus-matrix kernels (one per dataset per repetition, built by Consensus Clustering)
- Single-dataset kernel k-means clusterings (K=6) and KLIC integrative clustering with per-dataset kernel weights
- CSV table of ARI values per setting (A, B, C, D individually vs A+B+C+D) and average kernel weights per dataset, reproducing Fig. 2

## How to apply

Using the klic R package, generate 100 repetitions of four simulated datasets (A, B, C, D), each with 300 observations divided into 6 equal clusters sampled from bivariate normal distributions with a shared separation parameter s and an identity covariance matrix. For each dataset and repetition, build a consensus-matrix kernel via Consensus Clustering (Monti et al. 2003) as implemented in klic. Run kernel k-means on each individual consensus matrix with K=6 to obtain single-dataset baselines, then run KLIC on the four combined kernels with K=6 to obtain the integrative clustering and the learned per-dataset kernel weights. Compute the Adjusted Rand Index against the true labels for every baseline and each KLIC result, aggregate over the 100 repetitions, and write a CSV table of ARIs (A, B, C, D individually vs A+B+C+D) alongside average kernel weights, reproducing Fig. 2. Rationale: because all four datasets share equal dispersion and information content, correct behavior is a higher combined ARI and kernel weights divided equally across the four datasets.

## Related tools

- **klic** (Generates the simulated similar datasets, builds consensus-matrix kernels via Consensus Clustering, and performs Kernel Learning Integrative Clustering with learned kernel weights) — https://www.synapse.org/#!Synapse:syn2468297/wiki/64259
- **coca** (Companion R package from the same authors providing the consensus-clustering machinery against which KLIC is benchmarked in the simulation studies)
- **R** (Language in which the simulation scripts and 100-repetition benchmark are run)

## Examples

```
library(klic); library(coca)  # simulate 4 similar datasets (300 obs, 6 equal bivariate-normal clusters, separation s), build consensus kernels, run KLIC with K=6
```

## Evaluation signals

- ARI of KLIC on the combined A+B+C+D kernels is higher than the ARI of kernel k-means on each individual consensus matrix (A, B, C, or D alone)
- Average learned kernel weights are divided equally across the four datasets (~0.25 each), reflecting their equal dispersion and information content
- K is fixed to 6, matching the number of simulated clusters, in both single-dataset and integrative runs
- ARI values are aggregated over 100 repetitions rather than reported from a single run
- The CSV output table matches the structure of Fig. 2: ARI per setting plus average per-dataset kernel weights

## Limitations

- The benchmark targets datasets of equal dispersion/information content; the equal-weights outcome depends on this design and would not hold for noisy or unequal datasets
- Results are stochastic — conclusions require aggregation over the 100 repetitions, and single-repetition ARIs may not reflect the expected pattern
- Truth labels exist only because data are simulated; the ARI evaluation cannot be performed on real datasets without known subtypes

## Evidence

- [abstract] KLIC frames the challenge of combining clustering structures as a multiple kernel learning problem, in which: "KLIC frames the challenge of combining clustering structures as a multiple kernel learning problem, in which"
- [abstract] noisy-dataset down-weighting: "This allows the contribution of noisy datasets to be down-weighted relative to more informative datasets"
- [methods] kernel building via Consensus Clustering: "For each layer, one kernel is built using Consensus Clustering (Monti et al. 2003) and the most appropriate clustering algorithm."
- [abstract] simulation comparison of KLIC and COCA: "We compare the performances of KLIC and COCA in a variety of situations through simulation studies."
- [readme] README step 3 (KLIC on built kernels): "Integrative clustering is performed using Kernel Learning Integrative Clustering (Cabassi and Kirk, 2020) and the kernels built in the previous step."
