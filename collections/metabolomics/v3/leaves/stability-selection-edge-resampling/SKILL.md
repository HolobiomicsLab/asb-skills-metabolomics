---
name: stability-selection-edge-resampling
description: Use when building differential metabolomics networks with DNEA/Filigree
  where features outnumber samples (p >> n), and only resampling-stable edges should
  be retained after lambda tuning or approximation.
when_to_use_negative:
- The dataset is small (fewer than ~500 samples per group) and you still want BIC-tuned
  lambda rather than the Eq. 3 approximation — the BIC protocol requires large cohorts
- The analysis goal is pathway enrichment over curated databases (e.g. KEGG), not
  data-driven edge inference — DNEA's FELLA comparison showed only 19 of 174 TEDDY
  metabolites were KEGG-mappable
- You only need marginal correlations — those networks are denser and include indirect
  correlations that confound biologically relevant interactions
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: DNEA R package
  role: Provides stabilitySelection(), BICtune(), getNetworks(), and filterNetworks()
    for resampling-based edge stability selection on GLASSO partial-correlation networks
  repo: http://www.github.com/Karnovsky-Lab/DNEA/
- name: BiocParallel
  role: Parallel backend (BPPARAM) used to distribute the computationally intensive
    BICtune and stabilitySelection resampling steps across cores
- name: Filigree
  role: Earlier Java implementation whose defaults define the alternative configuration
    (aprox=TRUE lambda, 500 resampling reps without subsampling) compared against
    the DNEA R workflow
- name: R
  role: Runtime environment; the DNEA package is installed via BiocManager::install("DNEA")
  repo: http://www.github.com/Karnovsky-Lab/DNEA/
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1186/s12859-024-05994-1
    title: 'DNEA: an R package for fast and versatile data-driven network analysis
      of metabolomics data'
  source_content_hash: sha256:cb06870263d674b4aa9026238e581fd98c5d14bfa0a3c8f69fdc2efac80fd73b
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BiocParallel
  - DNEA R package
  - Filigree
  - R
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6923076923076923
n_claims: 13
---

# stability-selection-edge-resampling

## Summary

A resampling-based stability selection step in the DNEA R package that repeatedly re-estimates GLASSO partial-correlation networks across subsampled or bootstrapped data to identify metabolite-metabolite edges that are robust, controlling false positive edge inclusion under the p >> n regime.

## When to use

Use when building differential metabolomics networks with DNEA/Filigree where features outnumber samples (p >> n), and only resampling-stable edges should be retained after lambda tuning or approximation.

## When NOT to use

- The dataset is small (fewer than ~500 samples per group) and you still want BIC-tuned lambda rather than the Eq. 3 approximation — the BIC protocol requires large cohorts
- The analysis goal is pathway enrichment over curated databases (e.g. KEGG), not data-driven edge inference — DNEA's FELLA comparison showed only 19 of 174 TEDDY metabolites were KEGG-mappable
- You only need marginal correlations — those networks are denser and include indirect correlations that confound biologically relevant interactions

## Inputs

- DNEA object created by createDNEAobject() from raw peak intensities of two experimental conditions (e.g. IA case/control labels as a named factor vector)
- Per-group lambda penalty parameters from BICtune() or the approximation sqrt(ln(num features)/num samples)
- BiocParallel backend parameters (BPPARAM) for parallel resampling

## Outputs

- DNEA object annotated with edge-selection probabilities across nreps resamples
- Condition-specific and shared edge lists obtainable via edgeList() (e.g. 290 edges: 112 case-specific, 70 control-specific, 108 shared in one reported configuration)
- Filtered edge lists after filterNetworks() at a |pcor| >= 0.2 threshold

## How to apply

After preprocessing the TEDDY-style metabolomics data (drop metabolites missing in >30% of samples, median-impute the rest, regress out age/sex), run BICtune() with informed=TRUE and interval=1e-3 to optimize the lambda penalty per experimental group via BiocParallel when the dataset is large (~500+ samples per group). Then call stabilitySelection() — with subSample=TRUE and nreps=1000 when subsampling is desired, or nreps=500 without additional subsampling for the Filigree-default configuration — so that edge selection frequencies across resamples determine which GLASSO edges survive. For small, unbalanced cohorts the BIC protocol is unreliable; instead use getNetworks(aprox=TRUE) with lambda = sqrt(ln(num features)/num samples) before resampling. Finally, filter the networks with filterNetworks() at an absolute partial correlation cutoff (e.g. pcor = 0.166 in the vignette, or |pcor| >= 0.2 in the comparison study) before counting total and condition-specific edges.

## Related tools

- **DNEA R package** (Provides stabilitySelection(), BICtune(), getNetworks(), and filterNetworks() for resampling-based edge stability selection on GLASSO partial-correlation networks) — http://www.github.com/Karnovsky-Lab/DNEA/
- **BiocParallel** (Parallel backend (BPPARAM) used to distribute the computationally intensive BICtune and stabilitySelection resampling steps across cores)
- **Filigree** (Earlier Java implementation whose defaults define the alternative configuration (aprox=TRUE lambda, 500 resampling reps without subsampling) compared against the DNEA R workflow)
- **R** (Runtime environment; the DNEA package is installed via BiocManager::install("DNEA")) — http://www.github.com/Karnovsky-Lab/DNEA/

## Examples

```
TEDDYdat <- stabilitySelection(object = TEDDYdat, subSample = TRUE, nreps = 1000, BPPARAM = BP_plan)
```

## Evaluation signals

- Reproduced edge counts match the paper's reported values after |pcor| >= 0.2 filtering: 172 total edges with 139 differential for the BIC-tuned network and 70 total edges with 42 differential for the default network (vs. 70 total / 41 differential for the approximated-lambda configuration on the matched subset)
- The condition-specific plus shared edge counts sum to the total edge count (e.g. 112 + 70 + 108 = 290 edges)
- stabilitySelection was called with nreps matching the target configuration (1000 with subSample=TRUE, or 500 without additional subsampling for Filigree defaults)
- Lambda was chosen per experimental group either by BIC optimization (informed=TRUE, interval=1e-3) or by the sqrt(ln(features)/samples) approximation, and this choice is consistent with dataset size (~500+ samples per group for BIC)
- Network sparsity behaves as expected: total and condition-specific edges decrease as sample size decreases (e.g. T1D: 26,888 total edges at 100% vs 1,354 at 5% without subsampling)

## Limitations

- BIC tuning of lambda requires a large dataset (~500 or more samples per group), limiting applicability to smaller studies
- For small datasets the lambda approximation (Eq. 3) should be used instead of tuning, as it reduces false positive identification of unstable edges
- Subsampling of the larger group can reduce the total number of edges in the resulting network, as observed with the Framingham Heart Study metabolomics data
- As sample size decreases, both total and condition-specific edges decrease, producing sparser networks that may lose true interactions

## Evidence

- [methods] BICtune stabilitySelection call: "TEDDYdat <- stabilitySelection(object = TEDDYdat, subSample = TRUE, 
                               nreps = 1000, BPPARAM = BP_plan,"
- [results] BIC tuning without subsampling configuration: "The λ parameter was optimized using the BIC tuning protocol and stability selection was performed without additional subsampling"
- [discussion] BIC protocol sample-size requirement: "Utilizing the BIC protocol for network construction increases the sensitivity of DNEA, however, a large dataset is required (~ 500 or more samples per group)."
- [discussion] lambda approximation for small datasets: "For smaller datasets, an approximation of the λ parameter following Eq. 3 should be used, as it reduces false positive identification of unstable edges."
- [readme] rationale for regularization and stability selection: [evidence span withheld — anchor status: external_doc]
- [discussion] network filtering controls density: "the implementation of network filtering through the filterNetworks() function that aids interpretation by controlling the network density."
