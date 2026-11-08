---
name: gaussian-graphical-model-network-estimation
description: Use when a metabolomics/lipidomics feature table (raw peak intensities)
  has two condition groups with features outnumbering samples (p >> n) and you need
  regularized metabolite-metabolite partial correlation networks rather than pathway
  enrichment.
when_to_use_negative:
- Only one experimental condition is present — use the group's single-group tool CorrelationCalculator
  instead of the two-condition joint estimation
- Pathway-database enrichment is the goal and metabolites are well-annotated endogenous
  compounds mappable to KEGG — the data-driven GGM approach targets exogenous, secondary,
  and unknown metabolites that pathway analysis cannot map
- The dataset has far more samples than features so unregularized partial correlation
  estimation is feasible without the GLASSO penalty
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: DNEA R package
  role: Implements the GLASSO-based joint estimation of two-condition partial correlation
    networks via BICtune, stabilitySelection, getNetworks and filterNetworks
  repo: http://www.github.com/Karnovsky-Lab/DNEA/
- name: Filigree
  role: Prior Java implementation of the same DNEA workflow whose defaults (aprox
    lambda, nreps = 500) define the comparison configuration
- name: R
  role: Runtime language; the package runs on cloud/HPC systems using BiocParallel
    for parallelized tuning and stability selection
provenance:
  source_task_ids:
  - task_002
  - task_001
  source_papers:
  - doi: 10.1186/s12859-024-05994-1
    title: 'DNEA: an R package for fast and versatile data-driven network analysis
      of metabolomics data'
  source_content_hash: sha256:8a258f9b1f678471e37cf8861b4b668a6cf36943079140527d0b05cbb2f5227a
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Cytoscape
  - DNEA R package
  - Filigree
  - R
  - glasso
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9583333333333334
n_claims: 24
---

# gaussian-graphical-model-network-estimation

## Summary

Jointly estimate partial-correlation (Gaussian graphical model) networks for two experimental conditions from metabolomics peak-intensity data using GLASSO regularization with a tuned or approximated lambda penalty, as implemented in the DNEA R package.

## When to use

Use when a metabolomics/lipidomics feature table (raw peak intensities) has two condition groups with features outnumbering samples (p >> n) and you need regularized metabolite-metabolite partial correlation networks rather than pathway enrichment.

## When NOT to use

- Only one experimental condition is present — use the group's single-group tool CorrelationCalculator instead of the two-condition joint estimation
- Pathway-database enrichment is the goal and metabolites are well-annotated endogenous compounds mappable to KEGG — the data-driven GGM approach targets exogenous, secondary, and unknown metabolites that pathway analysis cannot map
- The dataset has far more samples than features so unregularized partial correlation estimation is feasible without the GLASSO penalty

## Inputs

- Raw (unnormalized) metabolomics peak-intensity matrix from two experimental conditions (e.g. Metabolomics Workbench ST001386 TEDDY data, 134 metabolites x 322 samples)
- Named factor vector of condition labels (e.g. 'DM:control' vs 'DM:case')
- Optionally covariate-adjusted residuals (age/sex linear-regression residuals shifted so each metabolite's minimum is zero)

## Outputs

- Jointly estimated condition-specific partial correlation networks (edgeList with shared and condition-specific edges)
- Edge selection probabilities from stability selection
- Filtered network retaining edges with |pcor| >= threshold (e.g. 0.2)
- Cytoscape-ready node and edge list files via getNetworkFiles()

## How to apply

Create the DNEA object with createDNEAobject() from raw peak intensities plus a named factor vector of group labels; the function log-transforms and auto-scales each group independently and checks eigenvalue/condition-number stability. Tune the GLASSO lambda penalty per group with BICtune(informed = TRUE, interval = 1e-3) under a BiocParallel backend, but only when each group has roughly 500 or more samples — for smaller datasets (e.g. the 134-metabolite TEDDY subset) pass aprox = TRUE to getNetworks(), which sets lambda to sqrt(ln(num features)/num samples). Run stabilitySelection() (subSample = TRUE with nreps = 1000 for unbalanced groups, or nreps = 500 without subsampling for balanced ones) to compute edge selection probabilities, then estimate the joint network with getNetworks(). Finally control network density by filtering with filterNetworks(), e.g. keeping only edges with absolute partial correlation >= 0.2 (the paper's vignette example uses pcor = 0.166).

## Related tools

- **DNEA R package** (Implements the GLASSO-based joint estimation of two-condition partial correlation networks via BICtune, stabilitySelection, getNetworks and filterNetworks) — http://www.github.com/Karnovsky-Lab/DNEA/
- **Filigree** (Prior Java implementation of the same DNEA workflow whose defaults (aprox lambda, nreps = 500) define the comparison configuration)
- **R** (Runtime language; the package runs on cloud/HPC systems using BiocParallel for parallelized tuning and stability selection)

## Examples

```
TEDDYdat <- getNetworks(object = TEDDYdat, aprox = TRUE, informed = TRUE); TEDDYdat_filtered <- filterNetworks(TEDDYdat, pcor = 0.2)
```

## Evaluation signals

- Reproducing the TEDDY analysis should yield 290 total edges (112 case-specific, 70 control-specific, 108 shared) before |pcor| filtering
- After filterNetworks(pcor = 0.2): BIC-tuned network should contain 172 total edges with 139 differential; default (approximated lambda) network 70 total with 41 differential
- createDNEAobject's eigenvalue/condition-number stability check should pass on the input before tuning
- BICtune should return per-group optimized lambda values minimizing BIC over the interval 1e-3 when informed = TRUE
- Exported node/edge list files and edgeList() output should parse correctly and match the reported edge counts

## Limitations

- BIC tuning protocol requires a large dataset (~500 or more samples per group); for smaller datasets the lambda approximation (Eq. 3) must be used instead
- Eq. 3 for the lambda approximation is not fully defined in the paper's discussion section
- No software version numbers are provided for DNEA or comparison tools, complicating exact reproduction
- Marginal-correlation alternatives (e.g. Metabox2) produce denser networks with indirect correlations that confound biological interpretation — the GGM approach controls for all other metabolites but still depends on regularization choices

## Evidence

- [readme] uses the GLASSO algorithm to estimate metabolite-metabolite partial correlations: "The package takes as input the raw peak intensity / concentrations for a set of features from two experimental conditions and uses the GLASSO algorithm to estimate metabolite-metabolite partial"
- [methods] TEDDYdat <- BICtune(object = TEDDYdat, informed = TRUE, interval = 1e-3, BPPARAM = BP_plan): "TEDDYdat <- BICtune(object = TEDDYdat, informed = TRUE, 
                    interval = 1e-3, BPPARAM = BP_plan)"
- [methods] stabilitySelection with subSample = TRUE, nreps = 1000: "TEDDYdat <- stabilitySelection(object = TEDDYdat, subSample = TRUE, 
                               nreps = 1000, BPPARAM = BP_plan,"
- [methods] getNetworks with aprox = TRUE: "TEDDYdat <- getNetworks(object = TEDDYdat, aprox = TRUE, informed = TRUE, "
- [intro] regularization needed when features exceed samples: "to obtain a partial correlation network when the number of metabolites measured in the dataset exceeds the number of the biological samples, regularization techniques need to be employed"
- [discussion] lambda approximation for smaller datasets reduces false positives: "For smaller datasets, an approximation of the λ parameter following Eq. 3 should be used, as it reduces false positive identification of unstable edges."
