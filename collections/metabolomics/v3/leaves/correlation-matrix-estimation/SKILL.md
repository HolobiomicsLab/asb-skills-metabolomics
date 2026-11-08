---
name: correlation-matrix-estimation
description: Use when evaluating how eigen-analysis-based Meff p-value adjustments
  behave in small-sample simulations (e.g. n=100) when the Pearson (PrsCo) matrix
  is replaced by a Spearman or DisCo correlation matrix over metabolite features.
when_to_use_negative:
- The analysis targets real experimental metabolomics data rather than the simulated
  n=100 replicate scenarios with a known true effective number of tests.
- The multiple-testing correction does not rely on eigen-analysis of a correlation
  matrix, so substituting Spearman for PrsCo is not meaningful.
- DisCo (distance correlation) is already the chosen matrix, since the skill is specifically
  about substituting Spearman and comparing against PrsCo.
edam_operation: http://edamontology.org/operation_3465
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: R
  role: Runs the eigen-analysis-based Meff estimators and computes the PrsCo, DisCo,
    and Spearman correlation matrices on simulated metabolite features for each replicate
  repo: github.com/KechrisLab/DisCoPad
- name: MATLAB
  role: Alternative environment for the example simulation scripts (one data sub-sample
    scenario) used to derive Meff estimates
  repo: github.com/KechrisLab/DisCoPad
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.3390/metabo15010028
    title: 'DisCo P-ad: Distance-Correlation-Based p-Value Adjustment Enhances Multiple
      Testing Corrections for Metabolomics'
  source_content_hash: sha256:894b174c27303648419d25bf819f5b56382e59d756a42ceafac4c2e7e90f6993
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MATLAB
  - R
  edam_operation: http://edamontology.org/operation_3465
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# correlation-matrix-estimation

## Summary

Compute and substitute alternative correlation matrices (Spearman rank, Pearson/PrsCo, distance correlation/DisCo) as inputs to eigen-analysis-based effective-number-of-tests (Meff) estimators in metabolomics multiple-testing simulations, and compare their RMSE against the true effective number of tests.

## When to use

Use when evaluating how eigen-analysis-based Meff p-value adjustments behave in small-sample simulations (e.g. n=100) when the Pearson (PrsCo) matrix is replaced by a Spearman or DisCo correlation matrix over metabolite features.

## When NOT to use

- The analysis targets real experimental metabolomics data rather than the simulated n=100 replicate scenarios with a known true effective number of tests.
- The multiple-testing correction does not rely on eigen-analysis of a correlation matrix, so substituting Spearman for PrsCo is not meaningful.
- DisCo (distance correlation) is already the chosen matrix, since the skill is specifically about substituting Spearman and comparing against PrsCo.

## Inputs

- Simulated metabolomics feature matrices from the n=100 sample-size scenario of the DisCoPad simulation (replicates)
- PrsCo (Pearson) and DisCo (distance correlation) reference matrices computed per replicate
- True effective number of tests for each replicate (RMSE reference)
- Eigen-analysis-based Meff estimator scripts from the DisCoPad 'Codes-Explained' folder (R and MATLAB)

## Outputs

- Spearman-based Meff estimates per replicate for each eigen-analysis-based estimator
- RMSE values of Spearman-, PrsCo-, and DisCo-based Meff estimates against the true effective number of tests
- spearman_meff_rmse_comparison.csv tabulating Meff estimates and RMSEs for Spearman, PrsCo, and DisCo side by side

## How to apply

Obtain the simulation scripts from the DisCoPad repository via its 'link-to-codes-and-data-objects' file, using Read-Me.txt to locate the script for the eigen-analysis-based Meff estimators. Load the target sample-size scenario (e.g. n=100) and compute the PrsCo (Pearson) and DisCo (distance correlation) reference matrices per replicate in R or MATLAB. Compute the Spearman rank correlation matrix on the same simulated metabolite features for each replicate, then substitute it in place of PrsCo in each eigen-analysis-based Meff estimator and re-derive Meff estimates for every replicate. Compute RMSE of the Spearman-based Meff estimates against the true effective number of tests alongside PrsCo- and DisCo-based RMSEs, and tabulate all three side by side. The accepted outcome mirrors the paper's Discussion claim: Spearman-based Meff estimates and RMSEs are relatively similar to PrsCo-based ones, whereas DisCo outperformed PrsCo with the highest margins in the n=100 scenario.

## Related tools

- **R** (Runs the eigen-analysis-based Meff estimators and computes the PrsCo, DisCo, and Spearman correlation matrices on simulated metabolite features for each replicate) — github.com/KechrisLab/DisCoPad
- **MATLAB** (Alternative environment for the example simulation scripts (one data sub-sample scenario) used to derive Meff estimates) — github.com/KechrisLab/DisCoPad

## Examples

```
Rscript meff_estimators.R --sample-size 100 --cor-matrices prsco,disco,spearman --out spearman_meff_rmse_comparison.csv
```

## Evaluation signals

- Spearman-based Meff estimates are relatively similar to the PrsCo-based estimates across all eigen-analysis-based estimators in the n=100 scenario (paper's Discussion claim).
- DisCo-based RMSE against the true effective number of tests is lower than both PrsCo- and Spearman-based RMSEs, with the highest margins in the n=100 scenario.
- spearman_meff_rmse_comparison.csv contains side-by-side Meff estimates and RMSEs for all three matrices (Spearman, PrsCo, DisCo) for every replicate.
- Every replicate uses the same simulated metabolite features to compute all three correlation matrices, ensuring apples-to-apples comparison.

## Limitations

- The closeness of Spearman- and PrsCo-based results holds for the n=100 simulation scenario; behavior at other sample size-to-feature ratios, response types, or metabolite groupings may differ.
- DisCo, not Spearman, was the matrix that outperformed PrsCo with the highest margins in the small-sample scenario, so Spearman substitution should not be expected to improve over Pearson.
- The repository provides example R and MATLAB codes only for one data sub-sample scenario of the simulation.

## Evidence

- [results] When Spearman's correlation matrix was substituted for the Pearson (PrsCo) matrix ... the results were relatively similar to the PrsCo-based results: [evidence span withheld — anchor status: not_found]
- [intro] Click on the 'link-to-codes-and-data-objects' file to access all codes and related items.: "Click on the "link-to-codes-and-data-objects" file to access all codes and related items."
- [readme] README instructions on locating example R and MATLAB codes: "Check out "Codes-Explained" folder to find all the example R and MATLAB codes (for one data sub-sample scenario) used for the simulation."
- [abstract] varying sample size-to-feature ratios, response types, and metabolite groupings: "Our study, including varying sample size-to-feature ratios, response types, and metabolite groupings, highlights the superior performance of the distance correlation."
