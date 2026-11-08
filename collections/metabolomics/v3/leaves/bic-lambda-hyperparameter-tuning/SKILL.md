---
name: bic-lambda-hyperparameter-tuning
description: Use when building GGM partial-correlation metabolite networks with DNEA
  on two-condition metabolomics data and the lambda penalty for glasso regularization
  must be set — either via BIC tuning (~500+ samples per group) or via the small-sample
  approximation.
when_to_use_negative:
- When either experimental group has fewer than ~500 samples — the article recommends
  the lambda approximation (Eq. 3) instead of BIC tuning, since BIC tuning requires
  a large dataset and the approximation reduces false positive unstable edges
- When the input is not a two-condition, approximately normal feature-intensity matrix
  (e.g. single-group data, where CorrelationCalculator rather than DNEA is appropriate)
edam_operation: http://edamontology.org/operation_0336
edam_topics:
- http://edamontology.org/topic_3334
- http://edamontology.org/topic_0091
tools:
- name: DNEA R package
  role: Provides BICtune() for per-group BIC lambda optimization, getNetworks(aprox=TRUE)
    for the small-sample lambda approximation, and stabilitySelection() for edge selection
    probabilities
  repo: http://www.github.com/Karnovsky-Lab/DNEA/
- name: BiocParallel
  role: Parallel backend (BPPARAM) over which the BIC tuning interval search is executed
- name: Filigree
  role: Prior Java implementation whose default lambda approximation (sqrt(ln(features)/samples))
    is reproduced by the aprox=TRUE code path
provenance:
  source_task_ids:
  - task_002
  - task_001
  source_papers:
  - doi: 10.1186/s12859-024-05994-1
    title: 'DNEA: an R package for fast and versatile data-driven network analysis
      of metabolomics data'
  source_content_hash: sha256:fa0cd9ea8ee159ee9c2974b26e19879b33d8b57cc85832d8b5e5c0f23a9774cd
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BiocParallel
  - Cytoscape
  - DNEA R package
  - Filigree
  - R
  - glasso
  edam_operation: http://edamontology.org/operation_0336
  edam_topics:
  - http://edamontology.org/topic_3334
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8666666666666667
n_claims: 15
---

# bic-lambda-hyperparameter-tuning

## Summary

Tunes the GLASSO regularization parameter (lambda) for DNEA joint network estimation by BIC optimization per experimental condition, using the BICtune() function with a BiocParallel backend, or falls back to the lambda approximation sqrt(ln(features)/samples) when datasets are too small for reliable BIC tuning.

## When to use

Use when building GGM partial-correlation metabolite networks with DNEA on two-condition metabolomics data and the lambda penalty for glasso regularization must be set — either via BIC tuning (~500+ samples per group) or via the small-sample approximation.

## When NOT to use

- When either experimental group has fewer than ~500 samples — the article recommends the lambda approximation (Eq. 3) instead of BIC tuning, since BIC tuning requires a large dataset and the approximation reduces false positive unstable edges
- When the input is not a two-condition, approximately normal feature-intensity matrix (e.g. single-group data, where CorrelationCalculator rather than DNEA is appropriate)

## Inputs

- DNEA object created with createDNEAobject() from raw peak intensities (e.g. Metabolomics Workbench ST001386 TEDDY metabolomics) with DM:control vs DM:case group labels
- Pre-processed feature table: metabolites missing in >30% of samples excluded, remaining values median-imputed, age/sex-adjusted residuals
- BiocParallel backend (BPPARAM) specification for parallel lambda search

## Outputs

- Per-group BIC-optimized lambda values stored in the DNEA object
- Glasso-estimated condition-specific and shared partial-correlation edge lists consumable by getNetworks(), stabilitySelection(), and filterNetworks()

## How to apply

Run BICtune(object, informed = TRUE, interval = 1e-3, BPPARAM = BP_plan) on the createDNEAobject() input (built from raw peak intensities and a named factor of group labels) so that a lambda is optimized per experimental group via BIC under a BiocParallel backend; then pass the optimized lambdas to getNetworks() and proceed to stabilitySelection(subSample = TRUE, nreps = 1000) for unbalanced groups. If each group has fewer than ~500 samples, instead use getNetworks(aprox = TRUE), which sets lambda = sqrt(ln(num features)/num samples) as in Filigree defaults, optionally followed by stabilitySelection with nreps = 500 without subsampling. Rationale: R1 regularization is required because features typically exceed samples (p >> n) in metabolomics, and the choice of lambda directly controls network density — in the paper's IA last-visit comparison, the BIC-tuned network yielded 172 total edges (139 differential) versus 70 total edges (41 differential) for the approximated-lambda network after |pcor| >= 0.2 filtering.

## Related tools

- **DNEA R package** (Provides BICtune() for per-group BIC lambda optimization, getNetworks(aprox=TRUE) for the small-sample lambda approximation, and stabilitySelection() for edge selection probabilities) — http://www.github.com/Karnovsky-Lab/DNEA/
- **BiocParallel** (Parallel backend (BPPARAM) over which the BIC tuning interval search is executed)
- **Filigree** (Prior Java implementation whose default lambda approximation (sqrt(ln(features)/samples)) is reproduced by the aprox=TRUE code path)

## Examples

```
TEDDYdat <- BICtune(object = TEDDYdat, informed = TRUE, interval = 1e-3, BPPARAM = BP_plan); TEDDYdat <- stabilitySelection(TEDDYdat, subSample = TRUE, nreps = 1000, BPPARAM = BP_plan)
```

## Evaluation signals

- BICtune() returns distinct informed lambda values per condition when informed = TRUE and interval = 1e-3
- In the reproduced IA last-visit comparison, BIC-tuned lambda networks yield 172 total edges with 139 differential after filterNetworks(pcor = 0.2), versus 70 total / 41 differential for the approximated-lambda configuration
- BIC-tuned configuration should produce a denser network than the approximated-lambda configuration on the same filtered data
- TEDDY T1D reference run (290 total edges: 70 control-specific, 112 case-specific, 108 shared) is reproducible when tuning and stability selection (subSample = TRUE, nreps = 1000) are applied in the documented order

## Limitations

- Utilizing the BIC protocol for network construction increases the sensitivity of DNEA, however, a large dataset is required (~ 500 or more samples per group)
- For smaller datasets the lambda approximation should be used instead, since BIC tuning on small unbalanced subsets can inflate unstable edges; the approximation itself reduces false positives but yields sparser networks
- No software version numbers were provided for DNEA or the comparison tools, so exact reproducibility of lambda search behavior across versions is unverified

## Evidence

- [methods] BICtune call in the methods vignette: "TEDDYdat <- BICtune(object = TEDDYdat, informed = TRUE, 
                    interval = 1e-3, BPPARAM = BP_plan)"
- [discussion] BIC tuning sample-size requirement and small-sample fallback: "Utilizing the BIC protocol for network construction increases the sensitivity of DNEA, however, a large dataset is required (~ 500 or more samples per group)."
- [intro] Motivation for regularization in GGM estimation: "to obtain a partial correlation network when the number of metabolites measured in the dataset exceeds the number of the biological samples, regularization techniques need to be employed"
- [readme] README: glasso-based joint estimation of two-condition networks: "takes as input the raw peak intensity / concentrations for a set of features from two experimental conditions and uses the GLASSO algorithm to estimate metabolite-metabolite partial correlations"
