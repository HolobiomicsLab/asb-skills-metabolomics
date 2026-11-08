---
name: convex-quadratic-programming-weight-estimation
description: Use when several N x N consensus/similarity kernel matrices over the
  same N samples must be combined via per-sample convex weights, and noisy datasets
  should be down-weighted relative to informative ones.
when_to_use_negative:
- Input is raw feature/expression data rather than precomputed N x N kernel matrices
  — build per-layer kernels via consensus clustering first (Kernel step of the pan-cancer
  workflow)
- The goal is integrative clustering on cluster memberships rather than kernels —
  COCA (Cluster-Of-Clusters Analysis) is the alternative strategy compared in the
  paper
- Only a single kernel/dataset is available and no down-weighting of noisy layers
  is needed — plain kernel k-means suffices
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: klic
  role: R package implementing Kernel Learning Integrative Clustering (KLIC), the
    multiple kernel learning approach whose weight-update step is the convex QP solved
    here
  repo: https://github.com/acabassi/klic-pancancer-analysis
- name: coca
  role: Companion R package providing the COCA alternative used for comparison in
    the simulation studies; not used inside the QP weight update itself
- name: R
  role: Runtime environment in which klic/coca are used to run kernel k-means and
    the weight-update QP and to save mkkm_model.rds
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btaa593
    title: Multiple kernel learning for integrative consensus clustering of omic datasets
  source_content_hash: sha256:fd284c509a7bddf2c9bf84697674535bb12aa75c96da8a0c31e0efc13c9886da
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
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# convex-quadratic-programming-weight-estimation

## Summary

Solves the sample-specific (observation-level) kernel-weight update of localized multiple kernel k-means as a convex quadratic program, alternating with kernel k-means on the weighted kernel sum. This mirrors KLIC's framing of integrative clustering as multiple kernel learning with weighted per-dataset contributions.

## When to use

Use when several N x N consensus/similarity kernel matrices over the same N samples must be combined via per-sample convex weights, and noisy datasets should be down-weighted relative to informative ones.

## When NOT to use

- Input is raw feature/expression data rather than precomputed N x N kernel matrices — build per-layer kernels via consensus clustering first (Kernel step of the pan-cancer workflow)
- The goal is integrative clustering on cluster memberships rather than kernels — COCA (Cluster-Of-Clusters Analysis) is the alternative strategy compared in the paper
- Only a single kernel/dataset is available and no down-weighting of noisy layers is needed — plain kernel k-means suffices

## Inputs

- M consensus/similarity kernel matrices (each N x N over the same N samples), e.g. one per omic layer built via consensus clustering (Monti et al. 2003)
- Optional true labels for a synthetic benchmark to compute ARI
- Initial sample-specific weight matrix theta (N x M) with rows summing to 1

## Outputs

- Cluster assignment vector over the N samples
- N x M sample-specific kernel weight matrix with each row a valid convex weight vector summing to 1
- Fitted model object (e.g. mkkm_model.rds) with recorded ARI against true labels

## How to apply

Load the M kernel (consensus) matrices, each N x N over the same N samples, and initialise the weights theta_im so each sample's weight vector sums to 1 over the M kernels. Run the alternating two-step optimisation: with weights fixed, perform kernel k-means on the convex combination sum_m theta_im K_m; with assignments fixed, minimise the objective over theta_im, which reduces to a convex quadratic program subject to the simplex constraint sum_m theta_im = 1 for each sample. Iterate until cluster assignments and weights converge, then save the cluster assignments and the N x M weight matrix (e.g. mkkm_model.rds). The rationale, as in KLIC (Cabassi and Kirk 2020), is that weighting lets the contribution of noisy datasets be down-weighted relative to more informative ones. When kernels come from consensus clustering per omic layer (as in the pan-cancer workflow: preprocessing -> per-layer kernels -> KLIC), each layer contributes one kernel built with the most appropriate clustering algorithm.

## Related tools

- **klic** (R package implementing Kernel Learning Integrative Clustering (KLIC), the multiple kernel learning approach whose weight-update step is the convex QP solved here) — https://github.com/acabassi/klic-pancancer-analysis
- **coca** (Companion R package providing the COCA alternative used for comparison in the simulation studies; not used inside the QP weight update itself)
- **R** (Runtime environment in which klic/coca are used to run kernel k-means and the weight-update QP and to save mkkm_model.rds)

## Evaluation signals

- Each row of the fitted N x M weight matrix is a valid convex weight vector: non-negative and summing to 1 over the M kernels
- Adjusted Rand Index (ARI) of cluster assignments against true labels exceeds the single-kernel baseline and lies in a plausible range
- Cluster assignments and weights converge (stable across alternating iterations) before saving the model
- Weight matrix (N x M) has the correct dimensions matching the N samples and M input kernels
- Down-weighting behaviour: noisy datasets receive lower weights than informative ones

## Limitations

- Requires all M kernels to be defined over the same N samples (same sample ordering/IDs)
- Two-step alternating optimisation means results depend on initialisation of weights and cluster assignments; iterate until convergence
- KLIC's properties were characterised through simulation studies and the Hoadley et al. (2014) pan-cancer application; behaviour on other data types should be validated separately

## Evidence

- [abstract] KLIC frames the challenge of combining clustering structures as a multiple kernel learning problem: "KLIC frames the challenge of combining clustering structures as a multiple kernel learning problem, in which"
- [abstract] Noisy datasets down-weighted: "This allows the contribution of noisy datasets to be down-weighted relative to more informative datasets"
- [readme] Per-layer kernel construction: "**Kernel**. For each layer, one kernel is built using Consensus Clustering (Monti et al. 2003) and the most appropriate clustering algorithm."
- [readme] KLIC consumes the per-layer kernels: "**KLIC**. Integrative clustering is performed using Kernel Learning Integrative Clustering (Cabassi and Kirk, 2020) and the kernels built in the previous step."
- [abstract] Comparison with COCA via simulations: "We compare the performances of KLIC and COCA in a variety of situations through simulation studies."
