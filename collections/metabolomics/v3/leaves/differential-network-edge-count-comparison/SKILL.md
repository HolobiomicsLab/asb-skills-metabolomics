---
name: differential-network-edge-count-comparison
description: Use when a two-condition metabolomics network has been estimated under
  alternative lambda/regularization settings and you must tabulate total vs case-specific
  vs control-specific vs shared edges to judge tuning impact.
when_to_use_negative:
- The dataset is a single experimental condition — DNEA performs joint network estimation
  across two groups, so differential edge counts are undefined
- The goal is pathway or module enrichment (NetGSA) rather than comparing network
  reconstruction settings
- Features/samples have not been preprocessed (metabolites missing in >30% of samples
  not excluded, median imputation and age/sex regression residuals not applied), making
  edge counts incomparable to the reference results
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_2269
tools:
- name: DNEA R package
  role: Builds the two-condition GLASSO differential networks, provides BICtune(),
    stabilitySelection(), getNetworks(), filterNetworks(), and edgeList() for counting
    edges under both lambda configurations
  repo: http://www.github.com/Karnovsky-Lab/DNEA/
- name: BiocParallel
  role: Parallel backend (BPPARAM) for the computationally intensive BICtune() lambda
    optimization and stabilitySelection() resampling steps
- name: Filigree
  role: Defines the baseline defaults being reproduced — the lambda approximation
    (aprox=TRUE) and default stability-selection settings constitute Configuration
    B
  repo: https://metscape.med.umich.edu/filigree.html
- name: R
  role: Runtime environment in which the DNEA package workflow and edge counting are
    executed
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1186/s12859-024-05994-1
    title: 'DNEA: an R package for fast and versatile data-driven network analysis
      of metabolomics data'
  source_content_hash: sha256:c66b2a0d42401475fa336239d5c70f7f5db609a7627b4926f47e8290516a0a47
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BiocParallel
  - DNEA R package
  - Filigree
  - R
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8666666666666667
n_claims: 15
---

# differential-network-edge-count-comparison

## Summary

Quantifies and compares total and condition-specific (differential) edge counts between two DNEA network configurations — BIC-tuned lambda versus the approximated lambda (sqrt(ln(num features)/num samples)) Filigree default — after filtering to |partial correlation| >= 0.2.

## When to use

Use when a two-condition metabolomics network has been estimated under alternative lambda/regularization settings and you must tabulate total vs case-specific vs control-specific vs shared edges to judge tuning impact.

## When NOT to use

- The dataset is a single experimental condition — DNEA performs joint network estimation across two groups, so differential edge counts are undefined
- The goal is pathway or module enrichment (NetGSA) rather than comparing network reconstruction settings
- Features/samples have not been preprocessed (metabolites missing in >30% of samples not excluded, median imputation and age/sex regression residuals not applied), making edge counts incomparable to the reference results

## Inputs

- Preprocessed DNEA object (createDNEAobject() built from raw peak intensities with case/control group labels as a named factor vector)
- BIC-optimized per-group lambda values (Configuration A)
- Approximated lambda = sqrt(ln(num features)/num samples) (Configuration B)
- Partial correlation threshold (0.2) for filterNetworks()

## Outputs

- Tabulated edge counts per configuration: total, condition-specific (case-only, control-only), and shared edges
- Edge lists from edgeList() for each filtered network
- Side-by-side comparison of BIC-tuned vs default-lambda network density

## How to apply

Estimate the differential network twice with the DNEA R package on the same preprocessed input: Configuration A uses BICtune() with informed=TRUE and interval=1e-3 (via BiocParallel) followed by stabilitySelection(subSample=TRUE, nreps=1000) and getNetworks() with the BIC-optimized per-group lambda; Configuration B uses the Filigree-default lambda approximation via getNetworks(aprox=TRUE) with stabilitySelection(nreps=500) without sub-sampling. Filter both networks with filterNetworks() using an absolute partial correlation threshold of 0.2 (the article also demonstrates pcor=0.166). Extract each network's edge list with edgeList() and count total edges, edges specific to each condition, and edges shared across conditions. The rationale is that edge counts directly reveal how lambda tuning versus approximation controls network sparsity and false-positive differential edges.

## Related tools

- **DNEA R package** (Builds the two-condition GLASSO differential networks, provides BICtune(), stabilitySelection(), getNetworks(), filterNetworks(), and edgeList() for counting edges under both lambda configurations) — http://www.github.com/Karnovsky-Lab/DNEA/
- **BiocParallel** (Parallel backend (BPPARAM) for the computationally intensive BICtune() lambda optimization and stabilitySelection() resampling steps)
- **Filigree** (Defines the baseline defaults being reproduced — the lambda approximation (aprox=TRUE) and default stability-selection settings constitute Configuration B) — https://metscape.med.umich.edu/filigree.html
- **R** (Runtime environment in which the DNEA package workflow and edge counting are executed)

## Examples

```
TEDDYdat <- BICtune(object = TEDDYdat, informed = TRUE, interval = 1e-3, BPPARAM = BP_plan); TEDDYdat <- stabilitySelection(object = TEDDYdat, subSample = TRUE, nreps = 1000, BPPARAM = BP_plan)
```

## Evaluation signals

- BIC-tuned network counts must match the reported reference: 172 total edges with 139 differential edges after |pcor| >= 0.2 filtering
- Default (approximated lambda) network counts must match: 70 total edges with 41 differential edges after |pcor| >= 0.2 filtering
- For each configuration, condition-specific + shared edges must sum exactly to the total edge count (e.g., 112 case-specific + 70 control-specific + 108 shared = 290 in the unfiltered IA analysis)
- The BIC-tuned configuration should yield a denser network than the approximated-lambda default on the same filtered input; if the ordering inverts, check lambda assignment per group
- Edge lists from edgeList() must contain only edges meeting the absolute partial correlation threshold used in filterNetworks()

## Limitations

- The BIC tuning protocol requires a large dataset (~500 or more samples per group); for smaller datasets the lambda approximation (Eq. 3) should be used instead, as it reduces false positive identification of unstable edges
- Subsampling of the larger group reduces the total number of edges (observed in the Framingham Heart Study experiments), so edge counts are sensitive to subsampling choices during stability selection
- Small, unbalanced case-control subsets (e.g., the TEDDY IA last-visit subset) produce sparse networks, and counts decrease further as sample size drops (T1D arm: 26,888 total edges at 100% samples vs 1,354 at 5% without subsampling)

## Evidence

- [intro] the filterNetworks() function is available to filter networks based on edge strength (percentile or partial correlation threshold) and highlight the most biologically relevant connections: "the filterNetworks() function is available to filter networks based on edge strength (percentile or partial correlation threshold) and highlight the most biologically relevant connections"
- [methods] BICtune code with interval parameter: "TEDDYdat <- BICtune(object = TEDDYdat, informed = TRUE, 
                    interval = 1e-3, BPPARAM = BP_plan)"
- [methods] stabilitySelection code: "TEDDYdat <- stabilitySelection(object = TEDDYdat, subSample = TRUE, 
                               nreps = 1000, BPPARAM = BP_plan,"
- [methods] The data were filtered to exclude metabolites that were missing in more than 30% of the samples.: "The data were filtered to exclude metabolites that were missing in more than 30% of the samples."
- [discussion] BIC protocol requires large dataset: "Utilizing the BIC protocol for network construction increases the sensitivity of DNEA, however, a large dataset is required (~ 500 or more samples per group)."
- [readme] README GLASSO input description: "The package takes as input the raw peak intensity / concentrations for a set of features from two experimental conditions and uses the GLASSO algorithm to estimate metabolite-metabolite partial"
