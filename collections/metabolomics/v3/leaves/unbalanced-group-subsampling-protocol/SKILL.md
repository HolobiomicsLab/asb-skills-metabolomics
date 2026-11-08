---
name: unbalanced-group-subsampling-protocol
description: Use when performing stability selection for differential GGM network
  estimation (DNEA/Filigree-style) on two-condition metabolomics data where group
  sizes are unbalanced and per-edge selection probabilities must weight the lambda
  penalty in final network estimation.
when_to_use_negative:
- Groups are already balanced — the article's TEDDY demo ran stability selection 'without
  additional subsampling' when subsampling was not needed, and unnecessary subsampling
  discards samples
- Datasets too small for the BIC tuning protocol (~500 or more samples per group required)
  — switch to the Eq. 3 lambda approximation instead of the tuned-lambda + subsampling
  pipeline
- Input is a single experimental condition with no group comparison — joint two-group
  estimation and its subsampling protocol do not apply
edam_operation: http://edamontology.org/operation_3744
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0602
tools:
- name: DNEA R package
  role: Provides stabilitySelection() with subSample and nreps arguments implementing
    the Filigree subsampling protocol, and selectionProbabilities() accessor
  repo: http://www.github.com/Karnovsky-Lab/DNEA/
- name: BiocParallel
  role: Embarrassingly-parallel backend (SnowParam/MulticoreParam with RNGseed) for
    the 1000 stability-selection replicates
- name: Filigree
  role: Original Java implementation whose subsampling protocol DNEA adopts for unbalanced
    groups
- name: R
  role: Runtime environment for the DNEA package and BiocParallel
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1186/s12859-024-05994-1
    title: 'DNEA: an R package for fast and versatile data-driven network analysis
      of metabolomics data'
  source_content_hash: sha256:3ecc52c858925228fb36980ba0af430d66e10384da01e83a9504ee9d5bec21c1
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BiocParallel
  - DNEA R package
  - Filigree
  - R
  edam_operation: http://edamontology.org/operation_3744
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7857142857142857
n_claims: 14
---

# unbalanced-group-subsampling-protocol

## Summary

A subsampling scheme inside the DNEA stabilitySelection() step that equalizes group representation when the two experimental conditions are unbalanced (e.g., 50 DM:case vs 272 DM:control TEDDY samples), so per-edge selection probabilities for GGM regularization reflect both groups fairly.

## When to use

Use when performing stability selection for differential GGM network estimation (DNEA/Filigree-style) on two-condition metabolomics data where group sizes are unbalanced and per-edge selection probabilities must weight the lambda penalty in final network estimation.

## When NOT to use

- Groups are already balanced — the article's TEDDY demo ran stability selection 'without additional subsampling' when subsampling was not needed, and unnecessary subsampling discards samples
- Datasets too small for the BIC tuning protocol (~500 or more samples per group required) — switch to the Eq. 3 lambda approximation instead of the tuned-lambda + subsampling pipeline
- Input is a single experimental condition with no group comparison — joint two-group estimation and its subsampling protocol do not apply

## Inputs

- DNEA object carrying metabolomics expression data (e.g. TEDDY data with BIC-tuned lambda 0.05072355)
- group_labels factor aligned to sample order (levels: 'DM:control', 'DM:case')
- BiocParallel backend configuration (workers, RNGseed)

## Outputs

- Per-edge selection probabilities for every metabolite-metabolite pair, values in [0, 1]
- Edge-wise rho table: rho = lambda * (1 / (1e-04 + selection.probability)) consumed by getNetworks()
- selection_probabilities.csv (edge-wise selection probability and rho table)

## How to apply

When group sample sizes differ substantially (here 50 DM:case vs 272 DM:control TEDDY metabolomics samples), call stabilitySelection() with subSample = TRUE and nreps = 1000: each replicate randomly subsamples the data using the Filigree-derived protocol that uses nearly all samples from the smaller group and matches representation from the larger group, then fits one GLASSO-based GGM per replicate. Parallelize replicates with a BiocParallel backend (SnowParam/MulticoreParam, workers = 2, RNGseed = 417 for reproducibility). Retrieve per-edge selection probabilities — the estimated probability that each metabolite-metabolite interaction is recovered in a randomly sampled data subset — via selectionProbabilities(). These probabilities then re-weight the regularization parameter per edge as rho = lambda * (1 / (1e-04 + selection.probability)) for the final getNetworks() estimation step. Rationale: resampling-based stability selection combined with subsampling avoids false-positive edges in the p >> n regime and prevents the majority group from dominating edge recovery.

## Related tools

- **DNEA R package** (Provides stabilitySelection() with subSample and nreps arguments implementing the Filigree subsampling protocol, and selectionProbabilities() accessor) — http://www.github.com/Karnovsky-Lab/DNEA/
- **BiocParallel** (Embarrassingly-parallel backend (SnowParam/MulticoreParam with RNGseed) for the 1000 stability-selection replicates)
- **Filigree** (Original Java implementation whose subsampling protocol DNEA adopts for unbalanced groups)
- **R** (Runtime environment for the DNEA package and BiocParallel)

## Examples

```
TEDDYdat <- stabilitySelection(object = TEDDYdat, subSample = TRUE, nreps = 1000, BPPARAM = BP_plan)
```

## Evaluation signals

- selection_probabilities.csv exists and contains a selection-probability column for every metabolite pair evaluated
- All selection probability values lie within [0, 1]
- Re-running with RNGseed = 417 reproduces identical probability values
- Per-edge rho values match rho = lambda * (1 / (1e-04 + selection.probability)) with lambda = 0.05072355
- Subsampling uses nearly all data from the smaller group (DM:case, n = 50) in each replicate

## Limitations

- For unbalanced groups the protocol subsamples the larger group, which reduces effective sample size and can yield sparser networks (e.g., TEDDY edges dropped from 26,888 at 100% to 1354 at 5% sampling)
- BIC tuning of the lambda feeding the weighting requires a large dataset (~500 or more samples per group); smaller studies should use the Eq. 3 lambda approximation
- Results depend on the RNGseed for exact reproducibility

## Evidence

- [methods] TEDDYdat <- stabilitySelection(object = TEDDYdat, subSample = TRUE, nreps = 1000, BPPARAM = BP_plan,: "TEDDYdat <- stabilitySelection(object = TEDDYdat, subSample = TRUE, 
                               nreps = 1000, BPPARAM = BP_plan,"
- [intro] Regularization, through both a penalty parameter and data resampling techniques: [evidence span withheld — anchor status: not_found]
- [discussion] In previous experiments using the metabolomics data from the Framingham heart study, subsampling of the larger group reduced the total number of edges: "In previous experiments using the metabolomics data from the Framingham heart study, subsampling of the larger group reduced the total number of edges"
- [readme] a novel method for stability selection, allow us to overcome this challenge: "R1 regularization, in combination with a novel method for stability selection, allow us to overcome this challenge"
