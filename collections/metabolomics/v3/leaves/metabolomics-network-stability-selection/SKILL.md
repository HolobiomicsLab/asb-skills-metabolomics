---
name: metabolomics-network-stability-selection
description: Use when building metabolite-metabolite networks from two-condition metabolomics
  data with p >> n, after BIC-tuning lambda, and you need per-edge selection probabilities
  to reweight regularization before getNetworks().
when_to_use_negative:
- When the dataset has fewer than ~500 samples per group and lambda cannot be BIC-tuned
  — the article recommends using the lambda approximation (Eq. 3) instead, and the
  tuning+selection protocol's sensitivity depends on large samples
- When analyzing a single experimental group — DNEA performs joint estimation across
  two conditions, and the subsampling protocol here is designed to balance two unbalanced
  groups
- When the input is raw peak intensities that have not passed upstream DNEA pre-processing
  (missing-value filtering >30%, median imputation, optional aggregateFeatures) and
  BIC tuning
edam_operation: http://edamontology.org/operation_3777
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0203
tools:
- name: DNEA R package
  role: Provides stabilitySelection(), selectionProbabilities(), and the downstream
    getNetworks() that consumes the probability-weighted regularization
  repo: http://www.github.com/Karnovsky-Lab/DNEA/
- name: Filigree
  role: Source of the subsampling protocol that equalizes group representation for
    unbalanced case/control samples
- name: BiocParallel
  role: Embarrassingly-parallel backend (SnowParam/MulticoreParam with RNGseed) for
    running the 1000 GGM replicate fits reproducibly
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1186/s12859-024-05994-1
    title: 'DNEA: an R package for fast and versatile data-driven network analysis
      of metabolomics data'
  source_content_hash: sha256:3e53e1f10b214871518d303c778534f5789d5c16dc65e607f69d73ee7c6d4dad
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BiocParallel
  - DNEA R package
  - Filigree
  - R
  edam_operation: http://edamontology.org/operation_3777
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 14
---

# metabolomics-network-stability-selection

## Summary

Stability-selection step of the DNEA workflow (STEP 2.2): fit repeated regularized GGMs on randomly sampled (and group-balanced) subsets to compute per-edge selection probabilities that weight the regularization parameter for final network estimation.

## When to use

Use when building metabolite-metabolite networks from two-condition metabolomics data with p >> n, after BIC-tuning lambda, and you need per-edge selection probabilities to reweight regularization before getNetworks().

## When NOT to use

- When the dataset has fewer than ~500 samples per group and lambda cannot be BIC-tuned — the article recommends using the lambda approximation (Eq. 3) instead, and the tuning+selection protocol's sensitivity depends on large samples
- When analyzing a single experimental group — DNEA performs joint estimation across two conditions, and the subsampling protocol here is designed to balance two unbalanced groups
- When the input is raw peak intensities that have not passed upstream DNEA pre-processing (missing-value filtering >30%, median imputation, optional aggregateFeatures) and BIC tuning

## Inputs

- DNEA object (e.g. TEDDYdat) holding TEDDY metabolomics expression data, T1Dmeta group labels (levels 'DM:control', 'DM:case'), and the BIC-tuned lambda from BICtune()
- BiocParallel backend configuration (BPPARAM, e.g. SnowParam/MulticoreParam with RNGseed = 417)
- stabilitySelection() parameters: subSample = TRUE, nreps = 1000

## Outputs

- Per-edge selection probabilities for every metabolite-metabolite interaction evaluated (via selectionProbabilities())
- Edge-wise rho table: rho = lambda * (1 / (1e-04 + selection.probability)) per edge
- selection_probabilities.csv with probability columns for all metabolite pairs

## How to apply

Load the DNEA object carrying the BIC-tuned lambda (e.g. 0.05072355 for the TEDDY metabolomics data), configure a BiocParallel backend (SnowParam/MulticoreParam with workers = 2, RNGseed = 417) for reproducible parallel replication, and call stabilitySelection() with subSample = TRUE and nreps = 1000. Because the TEDDY groups are unbalanced (50 DM:case vs. 272 DM:control), subsampling—adopted from Filigree—equalizes group representation by using nearly all data from the smaller group while fitting one GGM per replicate on random subsets of the larger group. Each replicate yields edge inclusion indicators; aggregating over the 1000 replicates gives, via selectionProbabilities(), the estimated probability that each potential metabolite-metabolite interaction is recovered in a randomly sampled data subset. Finally compute rho = lambda * (1 / (1e-04 + selection.probability)) per edge; these selection-probability-weighted penalties (Eq 2.2) are consumed by getNetworks() during final network estimation.

## Related tools

- **DNEA R package** (Provides stabilitySelection(), selectionProbabilities(), and the downstream getNetworks() that consumes the probability-weighted regularization) — http://www.github.com/Karnovsky-Lab/DNEA/
- **Filigree** (Source of the subsampling protocol that equalizes group representation for unbalanced case/control samples)
- **BiocParallel** (Embarrassingly-parallel backend (SnowParam/MulticoreParam with RNGseed) for running the 1000 GGM replicate fits reproducibly)

## Examples

```
TEDDYdat <- stabilitySelection(object = TEDDYdat, subSample = TRUE, nreps = 1000, BPPARAM = BP_plan)
```

## Evaluation signals

- selection_probabilities.csv exists and contains a selection-probability column for every metabolite pair evaluated
- All selection probability values lie within [0, 1]
- Re-running with RNGseed = 417 reproduces identical probability values (bitwise reproducibility)
- The computed rho = lambda * (1 / (1e-04 + selection.probability)) matches for every edge, using the BIC-tuned lambda (e.g. 0.05072355)
- 1000 replicates were fitted per the recommended protocol, one GGM per replicate

## Limitations

- The BIC tuning protocol feeding this step requires a large dataset (~500 or more samples per group), limiting applicability to smaller studies
- For smaller datasets, the lambda approximation (Eq. 3) should be used instead of tuning, as tuning reduces sensitivity of false-positive unstable edge identification in that regime
- Subsampling of the larger group reduces the total number of edges recovered (sparser networks as sample fraction decreases)

## Evidence

- [methods] In the stabilitySelection() step, the samples for each group are randomly sampled and a regularized model is fitted for each replicate (1000 replicates recommended) to calculate a selection probability for every potential metabolite-metabolite interaction: [evidence span withheld — anchor status: not_found]
- [methods] TEDDYdat <- stabilitySelection(object = TEDDYdat, subSample = TRUE, nreps = 1000, BPPARAM = BP_plan,: "TEDDYdat <- stabilitySelection(object = TEDDYdat, subSample = TRUE, 
                               nreps = 1000, BPPARAM = BP_plan,"
- [methods] because the groups are unbalanced (50 DM:case vs. 272 DM:control samples), the subsampling protocol from Filigree equalizes group representation: [evidence span withheld — anchor status: not_found]
- [intro] DNEA performs joint network estimation across two experimental groups, taking advantage of all available samples. Regularization, through both a penalty parameter and data resampling techniques, are: [evidence span withheld — anchor status: not_found]
- [discussion] Utilizing the BIC protocol for network construction increases the sensitivity of DNEA, however, a large dataset is required (~ 500 or more samples per group): "Utilizing the BIC protocol for network construction increases the sensitivity of DNEA, however, a large dataset is required (~ 500 or more samples per group)"
- [readme] R1 regularization, in combination with a novel method for stability selection, allow us to overcome this challenge by only keeping important metabolites in the model: [evidence span withheld — anchor status: external_doc]
