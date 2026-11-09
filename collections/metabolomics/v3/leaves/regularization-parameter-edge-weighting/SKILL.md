---
name: regularization-parameter-edge-weighting
description: Use when estimating a DNEA partial-correlation network for two experimental
  metabolomics groups and you need per-edge regularization weighting derived from
  stability-selection probabilities rather than a single global lambda.
when_to_use_negative:
- When stability selection was run without subsampling on severely unbalanced groups
  — the Filigree subsampling protocol is required to equalize group representation
  (e.g. 50 DM:case vs 272 DM:control samples)
- When lambda has not yet been tuned — the weighting multiplies the BIC-optimized
  lambda, so it cannot replace the BICtune() tuning step, which itself requires ~500
  or more samples per group
- When analyzing a single group — DNEA performs joint network estimation across two
  experimental conditions, and this weighting scheme is defined in that two-group
  workflow
edam_operation: null
edam_topics:
- http://edamontology.org/topic_3174
tools:
- name: DNEA R package
  role: Provides stabilitySelection(), selectionProbabilities(), and getNetworks()
    to compute selection probabilities and apply rho-weighted regularization
  repo: http://www.github.com/Karnovsky-Lab/DNEA/
- name: BiocParallel
  role: Embarrassingly-parallel backend (SnowParam/MulticoreParam) executing the 1000
    subsampled GGM replicates reproducibly via RNGseed
- name: R
  role: Language in which the stability-selection and rho-weighting computations are
    implemented and scripted
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1186/s12859-024-05994-1
    title: 'DNEA: an R package for fast and versatile data-driven network analysis
      of metabolomics data'
  source_content_hash: sha256:414438302df0608c7a84fd0f6c8e6c4566370884c92a399b2889259c9203d32e
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BiocParallel
  - DNEA R package
  - R
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# regularization-parameter-edge-weighting

## Summary

Computes per-edge selection probabilities via stability selection in the DNEA R package and converts them into an edge-specific regularization weight rho = lambda * (1/(1e-04 + selection.probability)) for final GGM network estimation. This suppresses unstable metabolite-metabolite interactions when features outnumber samples (p >> n).

## When to use

Use when estimating a DNEA partial-correlation network for two experimental metabolomics groups and you need per-edge regularization weighting derived from stability-selection probabilities rather than a single global lambda.

## When NOT to use

- When stability selection was run without subsampling on severely unbalanced groups — the Filigree subsampling protocol is required to equalize group representation (e.g. 50 DM:case vs 272 DM:control samples)
- When lambda has not yet been tuned — the weighting multiplies the BIC-optimized lambda, so it cannot replace the BICtune() tuning step, which itself requires ~500 or more samples per group
- When analyzing a single group — DNEA performs joint network estimation across two experimental conditions, and this weighting scheme is defined in that two-group workflow

## Inputs

- DNEA object carrying TEDDY-style metabolomics expression data, group_labels factor (levels 'DM:control', 'DM:case'), and a BIC-tuned lambda (e.g. 0.05072355)
- BiocParallel backend specification (SnowParam/MulticoreParam with workers = 2, RNGseed = 417)
- stabilitySelection() parameters: subSample = TRUE, nreps = 1000

## Outputs

- Per-edge selection probabilities (probability each metabolite-metabolite interaction is recovered in a random data subset) via selectionProbabilities()
- Edge-wise regularization weights rho = lambda * (1 / (1e-04 + selection.probability))
- selection_probabilities.csv containing selection-probability and rho columns for every metabolite pair evaluated

## How to apply

After BIC tuning has fixed lambda (e.g. 0.05072355 in the TEDDY demonstration), run stabilitySelection() with subSample = TRUE and nreps = 1000, supplying a BiocParallel backend (SnowParam/MulticoreParam, workers = 2, RNGseed = 417) for reproducible embarrassingly-parallel replication. Because groups can be unbalanced (TEDDY: 50 DM:case vs 272 DM:control samples), the Filigree-style subsampling protocol equalizes group representation, using nearly all data from the smaller group and fitting one GGM per replicate. Retrieve the per-edge selection probability — the estimated probability that each metabolite-metabolite interaction is recovered in a randomly sampled data subset — via selectionProbabilities(). Then compute rho = lambda * (1 / (1e-04 + selection.probability)) for every edge; these rho values weight the regularization parameter passed to getNetworks() during final network estimation. Write the edge-wise selection-probability and rho table to selection_probabilities.csv for downstream validation.

## Related tools

- **DNEA R package** (Provides stabilitySelection(), selectionProbabilities(), and getNetworks() to compute selection probabilities and apply rho-weighted regularization) — http://www.github.com/Karnovsky-Lab/DNEA/
- **BiocParallel** (Embarrassingly-parallel backend (SnowParam/MulticoreParam) executing the 1000 subsampled GGM replicates reproducibly via RNGseed)
- **R** (Language in which the stability-selection and rho-weighting computations are implemented and scripted)

## Examples

```
TEDDYdat <- stabilitySelection(object = TEDDYdat, subSample = TRUE, nreps = 1000, BPPARAM = BP_plan)
```

## Evaluation signals

- selection_probabilities.csv exists and contains a selection-probability column for every metabolite pair evaluated
- All selection-probability values lie within [0, 1]
- Every rho value equals lambda * (1 / (1e-04 + selection.probability)) with the BIC-tuned lambda (0.05072355 for the TEDDY demo)
- Re-running stabilitySelection() with RNGseed = 417 reproduces identical selection probabilities
- nreps = 1000 replicates were fitted, with one GGM per replicate and subsampling equalizing the 50 DM:case vs 272 DM:control group sizes

## Limitations

- Subsampling behavior depends on group balance; unbalanced designs (like TEDDY's 50 case vs 272 control) require the Filigree-style equalized subsampling protocol
- No software version numbers are provided for the DNEA R package in the article, complicating exact reproduction
- Selection probabilities depend on replicate count (1000 recommended); fewer replicates yield noisier probability estimates

## Evidence

- [methods] TEDDYdat <- stabilitySelection(object = TEDDYdat, subSample = TRUE, 
                               nreps = 1000, BPPARAM = BP_plan,: "TEDDYdat <- stabilitySelection(object = TEDDYdat, subSample = TRUE, 
                               nreps = 1000, BPPARAM = BP_plan,"
- [intro] DNEA performs joint network estimation across two experimental groups, taking advantage of all available samples. Regularization, through both a penalty parameter and data resampling techniques, are: "DNEA performs joint network estimation across two experimental groups, taking advantage of all available samples. Regularization, through both a penalty parameter"
- [readme] R1 regularization, in combination with a novel method for stability selection, allow us to overcome this challenge by only keeping important metabolites in the model.: [evidence span withheld — anchor status: external_doc]
- [methods] In the stabilitySelection() step, the samples for each group are randomly sampled and a regularized model is fitted for each replicate (1000 replicates recommended) to calculate a selection probability: [evidence span withheld — anchor status: not_found]
