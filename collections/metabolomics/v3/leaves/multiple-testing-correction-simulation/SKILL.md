---
name: multiple-testing-correction-simulation
description: Use when evaluating whether an alternative correlation matrix (e.g.,
  Spearman's rank correlation) improves or matches eigen-analysis-based Meff estimates
  in a simulation scenario such as a small sample size (n=100).
when_to_use_negative:
- The goal is a single real metabolomics dataset analysis rather than a simulation
  benchmark across replicates with a known true Meff
- The multiple testing correction procedure in use does not rely on eigen-analysis
  of a correlation matrix (e.g., Bonferroni-type corrections independent of correlation
  structure)
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0203
tools:
- name: R
  role: Language used to load the n=100 simulation scenario, compute Pearson/Spearman/DisCo
    matrices, and run the eigen-analysis-based Meff estimators
  repo: github.com/KechrisLab/DisCoPad
- name: MATLAB
  role: Alternative language (equivalent to R) for running the example simulation
    scripts in the Codes-Explained folder
  repo: github.com/KechrisLab/DisCoPad
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.3390/metabo15010028
    title: 'DisCo P-ad: Distance-Correlation-Based p-Value Adjustment Enhances Multiple
      Testing Corrections for Metabolomics'
  source_content_hash: sha256:be668fc344f08ca84111428f56ddf60de04906ddbb1a833f90e6360c52b15718
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MATLAB
  - R
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# multiple-testing-correction-simulation

## Summary

A simulation-based skill for benchmarking effective-number-of-tests (Meff) estimators: substitute alternative correlation matrices (Spearman, DisCo, PrsCo) into eigen-analysis-based multiple testing correction procedures and compare RMSE against the true effective number of tests, as done in the DisCo P-ad metabolomics study.

## When to use

Use when evaluating whether an alternative correlation matrix (e.g., Spearman's rank correlation) improves or matches eigen-analysis-based Meff estimates in a simulation scenario such as a small sample size (n=100).

## When NOT to use

- The goal is a single real metabolomics dataset analysis rather than a simulation benchmark across replicates with a known true Meff
- The multiple testing correction procedure in use does not rely on eigen-analysis of a correlation matrix (e.g., Bonferroni-type corrections independent of correlation structure)

## Inputs

- Simulation scripts and data objects from the DisCoPad repository (accessed via the 'link-to-codes-and-data-objects' file)
- Simulated metabolite feature matrices for a fixed sample-size scenario (e.g., n=100) across replicates
- True effective number of tests per replicate for RMSE calculation

## Outputs

- Meff estimates for each replicate under Spearman-, PrsCo-, and DisCo-based correlation matrices
- RMSE of each correlation-matrix-based Meff estimator against the true effective number of tests
- Side-by-side comparison table (e.g., spearman_meff_rmse_comparison.csv)

## How to apply

Obtain the simulation scripts and data objects from the DisCoPad repository via its 'link-to-codes-and-data-objects' file, and consult the 'Read-Me.txt' to identify which script implements the eigen-analysis-based Meff estimators. Load the target sample-size scenario (e.g., n=100) in R or MATLAB and compute the PrsCo (Pearson) and DisCo (distance correlation) reference matrices for each replicate. Compute the Spearman rank correlation matrix on the same simulated metabolite features for each replicate, then substitute it in place of PrsCo inside each eigen-analysis-based Meff estimator and re-derive the Meff estimates per replicate. Compute RMSE of each estimator's Meff estimates against the true effective number of tests across replicates, and tabulate Spearman, PrsCo, and DisCo side by side (e.g., writing spearman_meff_rmse_comparison.csv).

## Related tools

- **R** (Language used to load the n=100 simulation scenario, compute Pearson/Spearman/DisCo matrices, and run the eigen-analysis-based Meff estimators) — github.com/KechrisLab/DisCoPad
- **MATLAB** (Alternative language (equivalent to R) for running the example simulation scripts in the Codes-Explained folder) — github.com/KechrisLab/DisCoPad

## Examples

```
Rscript run_meff_estimators.R --correlation spearman --n 100 --out spearman_meff_rmse_comparison.csv
```

## Evaluation signals

- Spearman-based Meff estimates and RMSEs are relatively similar to the PrsCo-based ones, matching the paper's Discussion claim
- DisCo-based results outperform PrsCo with the highest margins in the n=100 scenario, serving as a sanity reference
- RMSE values are computed against the true effective number of tests across replicates, not just observed Meff
- The output CSV (spearman_meff_rmse_comparison.csv) tabulates Spearman, PrsCo, and DisCo columns for every replicate/estimator
- Scripts were located by following the repository's 'link-to-codes-and-data-objects' file and the 'Read-Me.txt' description of each script

## Limitations

- The repository's example R and MATLAB codes cover only one data sub-sample scenario of the full simulation, so other sample-size scenarios may require adaptation
- The similarity of Spearman- to PrsCo-based results is relative; DisCo was the superior performer, so Spearman substitution is not an improvement over distance correlation
- The skill was demonstrated for eigen-analysis-based Meff estimators and may not transfer to non-eigen-based p-value adjustment procedures

## Evidence

- [intro] Repository provides links to all codes and data-files used in the article: "Repository that contains links to all the codes and data-files used in the article."
- [intro] Example codes for one sub-sample scenario: "find all the example R and MATLAB codes (for one data sub-sample scenario) used for the simulation"
- [abstract] Superior performance of distance correlation across simulation conditions: "Our study, including varying sample size-to-feature ratios, response types, and metabolite groupings, highlights the superior performance of the distance correlation."
- [readme] README instructs to consult Read-Me.txt for script purpose: "Refer to the "Read-Me.txt" file to understand what each script accomplishes."
