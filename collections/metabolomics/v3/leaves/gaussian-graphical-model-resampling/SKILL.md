---
name: gaussian-graphical-model-resampling
description: Use when estimating partial-correlation metabolite-metabolite networks
  under p >> n with DNEA, after BIC tuning of lambda, and before getNetworks(), to
  stabilize edge inclusion via resampling.
when_to_use_negative:
- When analyzing a single experimental group — DNEA's stability selection is built
  for joint estimation across two conditions and subsampling equalization of two groups
- When the input is raw peak-intensity data that has not been pre-processed, feature-aggregated
  (aggregateFeatures), and lambda-tuned (BICtune) — selection probabilities would
  be conditioned on an untuned penalty
- When the goal is marginal correlation networks for exploratory co-occurrence — partial-correlation
  GGMs target direct associations and the resampling machinery is unnecessary overhead
edam_operation: http://edamontology.org/operation_3766
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0203
tools:
- name: DNEA R package
  role: Provides stabilitySelection(), selectionProbabilities(), and getNetworks()
    implementing the resampling-based stability selection and rho weighting
  repo: http://www.github.com/Karnovsky-Lab/DNEA/
- name: R
  role: Statistical programming environment in which the DNEA package runs, including
    on cloud/HPC systems
- name: Filigree
  role: Prior Java implementation whose subsampling protocol for unbalanced groups
    DNEA reuses
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1186/s12859-024-05994-1
    title: 'DNEA: an R package for fast and versatile data-driven network analysis
      of metabolomics data'
  source_content_hash: sha256:1a9826447522fc67795836b11617909a0b8f2ca3a9237173a3283e2968510310
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - DNEA R package
  - Filigree
  - R
  edam_operation: http://edamontology.org/operation_3766
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# gaussian-graphical-model-resampling

## Summary

Stability-selection step of the DNEA workflow that repeatedly refits GLASSO-based Gaussian graphical models on randomly subsampled data (1000 replicates via BiocParallel) to compute per-edge selection probabilities, which then weight the regularization parameter for final network estimation.

## When to use

Use when estimating partial-correlation metabolite-metabolite networks under p >> n with DNEA, after BIC tuning of lambda, and before getNetworks(), to stabilize edge inclusion via resampling.

## When NOT to use

- When analyzing a single experimental group — DNEA's stability selection is built for joint estimation across two conditions and subsampling equalization of two groups
- When the input is raw peak-intensity data that has not been pre-processed, feature-aggregated (aggregateFeatures), and lambda-tuned (BICtune) — selection probabilities would be conditioned on an untuned penalty
- When the goal is marginal correlation networks for exploratory co-occurrence — partial-correlation GGMs target direct associations and the resampling machinery is unnecessary overhead

## Inputs

- DNEA object carrying expression data and the BIC-tuned lambda (e.g. TEDDY metabolomics data with lambda = 0.05072355)
- Group labels factor aligned to sample order (levels: 'DM:control', 'DM:case') from T1Dmeta metadata
- BiocParallel backend configuration (e.g. SnowParam/MulticoreParam, workers = 2, RNGseed = 417)

## Outputs

- Per-edge selection probabilities (probability each metabolite-metabolite interaction is recovered across resampled subsets)
- Per-edge weighted regularization values rho = lambda * (1 / (1e-04 + selection.probability))
- Edge-wise selection-probability table written to selection_probabilities.csv

## How to apply

After BICtune() has fixed lambda (e.g. 0.05072355 for the TEDDY data), run stabilitySelection() with subSample = TRUE and nreps = 1000, parallelized through a BiocParallel backend (SnowParam/MulticoreParam with workers and RNGseed = 417 for reproducibility). Because groups can be unbalanced (TEDDY: 50 DM:case vs. 272 DM:control samples), the Filigree-style subsampling protocol equalizes group representation, using nearly all data from the smaller group and fitting one GGM per replicate. Each replicate's GLASSO fit yields, for every potential metabolite-metabolite interaction, the estimated probability that the edge is recovered in a randomly sampled data subset; retrieve these via selectionProbabilities(). Compute the per-edge weighting rho = lambda * (1 / (1e-04 + selection.probability)), which stabilitySelection() passes forward so getNetworks() applies stronger penalization to less-stable edges during final network estimation. Write the edge-wise selection-probability (and rho) table (e.g. selection_probabilities.csv) for validation and downstream inspection.

## Related tools

- **DNEA R package** (Provides stabilitySelection(), selectionProbabilities(), and getNetworks() implementing the resampling-based stability selection and rho weighting) — http://www.github.com/Karnovsky-Lab/DNEA/
- **R** (Statistical programming environment in which the DNEA package runs, including on cloud/HPC systems)
- **Filigree** (Prior Java implementation whose subsampling protocol for unbalanced groups DNEA reuses)

## Examples

```
TEDDYdat <- stabilitySelection(object = TEDDYdat, subSample = TRUE, nreps = 1000, BPPARAM = BP_plan)
```

## Evaluation signals

- selection_probabilities.csv exists and contains a selection-probability column for every metabolite pair evaluated
- All selection probability values lie within [0, 1]
- Re-running stabilitySelection() with RNGseed = 417 reproduces identical probability values
- 1000 replicates were completed and a fitted GGM exists per replicate (subSample = TRUE equalized the 50 DM:case vs. 272 DM:control groups)
- Computed rho values satisfy rho = lambda * (1 / (1e-04 + selection.probability)) per edge using the BIC-tuned lambda

## Limitations

- Stability selection resampling is computationally intensive; 1000 GGM fits per replicate set requires parallelization (BiocParallel) on HPC or cloud machines
- Selection probabilities are only meaningful after proper lambda tuning; the article notes the BIC tuning protocol itself requires ~500 or more samples per group, so smaller studies must rely on the Eq. 3 lambda approximation
- Subsampling of the larger group can reduce the total number of edges (observed with Framingham Heart Study metabolomics data), affecting edge density

## Evidence

- [methods] stabilitySelection: "TEDDYdat <- stabilitySelection(object = TEDDYdat, subSample = TRUE, 
                               nreps = 1000, BPPARAM = BP_plan,"
- [intro] Resampling rationale: [evidence span withheld — anchor status: not_found]
- [readme] p >> n problem: [evidence span withheld — anchor status: external_doc]
- [readme] Parallelization requirement: "DNEA is designed to work on high-performance or cloud-computing machines to utilize all available computing cores through parallelization"
