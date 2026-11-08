---
name: stability-selection-edge-probability-estimation
description: Use when building differential metabolomics/lipidomics networks (p >>
  n, two conditions) and you need robust edge retention before GLASSO network estimation,
  e.g. TEDDY case-vs-control data with 134 metabolites and unbalanced groups.
when_to_use_negative:
- Single-condition network construction without a case/control contrast — use CorrelationCalculator
  instead, which builds a data-driven network for one group
- Datasets where features cannot be treated as approximately normal intensity/concentration
  values (the package is built for metabolomics/lipidomics or similar -omics data)
- Datasets lacking enough samples for either BIC tuning (~500+ per group) or the lambda
  approximation rationale — edge probabilities will be unstable
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0203
tools:
- name: DNEA R package
  role: Provides stabilitySelection() which computes edge selection probabilities
    via resampling (subSample = TRUE, nreps = 1000) as part of the differential network
    workflow
  repo: http://www.github.com/Karnovsky-Lab/DNEA/
- name: BiocParallel
  role: Parallel backend (BPPARAM) used to run the computationally intensive stability
    selection resampling across cores
- name: glasso
  role: Underlying R1-regularized GLASSO partial-correlation algorithm whose edge
    selections are evaluated across resamples
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1186/s12859-024-05994-1
    title: 'DNEA: an R package for fast and versatile data-driven network analysis
      of metabolomics data'
  source_content_hash: sha256:baa86ea864d01fb746f048b45216c622f1a60c416cca0101e22b76a4df660ac8
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BiocParallel
  - Cytoscape
  - DNEA R package
  - R
  - glasso
  edam_operation: http://edamontology.org/operation_3927
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8666666666666667
n_claims: 15
---

# Stability selection edge probability estimation

## Summary

Estimates metabolite–metabolite edge selection probabilities by resampling (stability selection) in the DNEA R package, so that only edges reproducibly selected under R1-regularized GLASSO survive joint network estimation of two experimental groups.

## When to use

Use when building differential metabolomics/lipidomics networks (p >> n, two conditions) and you need robust edge retention before GLASSO network estimation, e.g. TEDDY case-vs-control data with 134 metabolites and unbalanced groups.

## When NOT to use

- Single-condition network construction without a case/control contrast — use CorrelationCalculator instead, which builds a data-driven network for one group
- Datasets where features cannot be treated as approximately normal intensity/concentration values (the package is built for metabolomics/lipidomics or similar -omics data)
- Datasets lacking enough samples for either BIC tuning (~500+ per group) or the lambda approximation rationale — edge probabilities will be unstable

## Inputs

- DNEA object created with createDNEAobject() from a metabolomics feature table (e.g. TEDDY T1D subset: 134 metabolites, 322 samples, ST001386/ST001636) plus T1Dmeta sample metadata with 'DM:control' vs 'DM:case' group labels
- Tuned lambda hyperparameter from BICtune(informed = TRUE, interval = 1e-3) or the approximation sqrt(ln(features)/samples) for groups with < 500 samples
- BiocParallel backend (BP param) for parallel execution

## Outputs

- DNEA object annotated with per-edge stability selection probabilities suitable for getNetworks() joint estimation
- Downstream: 290-edge joint network (70 control-specific, 112 case-specific, 108 shared for TEDDY) for consensus clustering

## How to apply

After data pre-processing (exclude metabolites missing in >30% of samples, median-impute the rest, adjust for covariates such as age and gender via regression residuals) and construction of the DNEA object with createDNEAobject() (which log-transforms and auto-scales each group independently), first tune the lambda hyperparameter with BICtune(informed = TRUE, interval = 1e-3) under a BiocParallel backend; if each group has fewer than ~500 samples, use the lambda approximation sqrt(ln(features)/samples) instead of BIC tuning. Then run stabilitySelection(subSample = TRUE, nreps = 1000, BPPARAM = ...) to compute edge selection probabilities; subsampling is specifically used when the two experimental groups are unbalanced in size. R1 regularization plus this novel stability-selection resampling overcomes the p >> n problem by keeping only important metabolites/edges in the model, reducing false-positive unstable edges before getNetworks() jointly estimates the two networks. The resulting edge probabilities feed into consensus clustering (clusterNet, tau = 0.5) and downstream NetGSA enrichment testing.

## Related tools

- **DNEA R package** (Provides stabilitySelection() which computes edge selection probabilities via resampling (subSample = TRUE, nreps = 1000) as part of the differential network workflow) — http://www.github.com/Karnovsky-Lab/DNEA/
- **BiocParallel** (Parallel backend (BPPARAM) used to run the computationally intensive stability selection resampling across cores)
- **glasso** (Underlying R1-regularized GLASSO partial-correlation algorithm whose edge selections are evaluated across resamples)

## Examples

```
BP_plan <- BiocParallel::MulticoreParam(); TEDDYdat <- BICtune(TEDDYdat, informed = TRUE, interval = 1e-3, BPPARAM = BP_plan); TEDDYdat <- stabilitySelection(TEDDYdat, subSample = TRUE, nreps = 1000, BPPARAM = BP_plan)
```

## Evaluation signals

- stabilitySelection was called with subSample = TRUE and nreps = 1000 (documented settings for unbalanced groups like TEDDY)
- A tuned lambda value exists from BICtune(informed = TRUE, interval = 1e-3), or the approximation sqrt(ln(features)/samples) was applied because each group has < 500 samples
- The joint network after downstream getNetworks() contains 290 total edges (70 control-specific, 112 case-specific, 108 shared) for the TEDDY reference dataset
- Subsequent clusterNet(tau = 0.5) yields 11 subnetworks with >= 5 metabolites testable by runNetGSA(min_size = 5), 8 significant at FDR < 0.05
- Parallel backend (BiocParallel BPPARAM) was supplied so the nreps resampling completed without serial fallback

## Limitations

- BIC tuning of lambda requires a large dataset (~500 or more samples per group); smaller studies must use the lambda approximation (Eq. 3), which 'is not fully defined' in the article section, increasing ambiguity
- As sample size decreases, both total and condition-specific edges decrease, producing sparser networks (e.g. T1D arm: 26,888 total edges at 100% samples vs 1354 at 5% without subsampling), so stability-selection outputs are sample-size dependent
- GGM-based stability selection is computationally intensive and is designed for high-performance or cloud machines with parallelization

## Evidence

- [methods] stability selection call: "TEDDYdat <- stabilitySelection(object = TEDDYdat, subSample = TRUE, 
                               nreps = 1000, BPPARAM = BP_plan,"
- [methods] BIC tuning: "TEDDYdat <- BICtune(object = TEDDYdat, informed = TRUE, 
                    interval = 1e-3, BPPARAM = BP_plan)"
- [discussion] lambda approximation rationale: "For smaller datasets, an approximation of the λ parameter following Eq. 3 should be used, as it reduces false positive identification of unstable edges."
- [discussion] BIC requires large datasets: "Utilizing the BIC protocol for network construction increases the sensitivity of DNEA, however, a large dataset is required (~ 500 or more samples per group)."
- [readme] regularization + stability selection rationale: [evidence span withheld — anchor status: external_doc]
- [results] sparsity with fewer samples: "As the number of samples in the dataset decreased, both the total number of edges as well as the number of edges specific to each condition decreased, resulting in a sparser network."
