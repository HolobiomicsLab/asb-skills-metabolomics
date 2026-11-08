---
name: rmse-aggregation-across-scenarios
description: Use when comparing multiple Meff estimation methods across a factorial
  grid of simulation scenarios against a gold standard, to quantify which method/matrix
  combination (e.g., DisCo vs PrsCo) yields the lowest RMSE per cell.
when_to_use_negative:
- The aim is raw per-replicate Meff comparison rather than scenario-level median aggregation
  and RMSE against a gold standard
- No permutation-based gold-standard Meff0 is available to serve as the reference
  for RMSE computation
- Input data are experimental (not simulated) p-values without the paired PrsCo/DisCo
  matrix structure required for the 2 x 5 x 3 x 3 cell grid
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_3517
- http://edamontology.org/topic_0091
tools:
- name: R
  role: Compute eigen-analysis-based Meff estimates, aggregate medians, and calculate
    RMSE for the simulation scenario grid
  repo: github.com/KechrisLab/DisCoPad
- name: MATLAB
  role: Example codes (Codes-Explained folder) for one data sub-sample scenario used
    in the simulation
  repo: github.com/KechrisLab/DisCoPad
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.3390/metabo15010028
    title: 'DisCo P-ad: Distance-Correlation-Based p-Value Adjustment Enhances Multiple
      Testing Corrections for Metabolomics'
  source_content_hash: sha256:a88e2d572ce70a7340f618b10f8a551a6fb0f7963cc263d9438f57278fe753b3
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MATLAB
  - R
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_3517
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# rmse-aggregation-across-scenarios

## Summary

Aggregates effective-number-of-tests (Meff) estimates from eigen-analysis-based p-value adjustment methods across simulation scenarios (matrix type, metabolite grouping, sample size) by taking per-cell medians, then computes RMSE of those medians against a permutation-based gold-standard Meff0 to rank methods such as DisCo P-ad versus PrsCo.

## When to use

Use when comparing multiple Meff estimation methods across a factorial grid of simulation scenarios against a gold standard, to quantify which method/matrix combination (e.g., DisCo vs PrsCo) yields the lowest RMSE per cell.

## When NOT to use

- The aim is raw per-replicate Meff comparison rather than scenario-level median aggregation and RMSE against a gold standard
- No permutation-based gold-standard Meff0 is available to serve as the reference for RMSE computation
- Input data are experimental (not simulated) p-values without the paired PrsCo/DisCo matrix structure required for the 2 x 5 x 3 x 3 cell grid

## Inputs

- Simulated metabolomics p-value sub-samples from the upstream simulation (3 sample sizes, e.g. n=100)
- PrsCo Pearson correlation matrices per grouping and sample size
- DisCo distance-correlation matrices per grouping and sample size
- Gold-standard permutation-based Meff0 estimates (from task_003)

## Outputs

- meff_rmse_results.csv with method, matrix type (PrsCo/DisCo), metabolite grouping, sample size, and RMSE columns covering all 2 x 5 x 3 x 3 cells
- Median-aggregated Meff estimates per method x matrix x grouping x sample-size cell

## How to apply

Load the simulated metabolomics p-value sub-samples together with the PrsCo (Pearson correlation) and DisCo (distance correlation) matrices produced upstream. For each of the five eigen-analysis-based p-value adjustment methods, compute Meff from the eigen-decomposition of the PrsCo matrix for each metabolite grouping (ungrouped, defined 8-pathway, random) and each of the three sample sizes; repeat using the DisCo matrix to implement the DisCo P-ad modification of each method. Aggregate Meff estimates across simulation replicates by taking the median per method x matrix x grouping x sample-size cell, then compute RMSE of the median Meff estimates against the gold-standard Meff0 for every cell. Write the full RMSE comparison table to meff_rmse_results.csv, preserving method, matrix type, grouping, and sample-size columns so downstream checks (e.g., DisCo yielding the smallest RMSE for Nyholt at n=100; grouping reducing RMSE versus ungrouped) can be verified.

## Related tools

- **R** (Compute eigen-analysis-based Meff estimates, aggregate medians, and calculate RMSE for the simulation scenario grid) — github.com/KechrisLab/DisCoPad
- **MATLAB** (Example codes (Codes-Explained folder) for one data sub-sample scenario used in the simulation) — github.com/KechrisLab/DisCoPad

## Examples

```
Rscript compute_meff_rmse.R --prsco prsco_matrices.rds --disco disco_matrices.rds --meff0 meff0_gold.rds --out meff_rmse_results.csv
```

## Evaluation signals

- Output table meff_rmse_results.csv exists and covers all 2 matrices x 5 methods x 3 groupings x 3 sample sizes cells
- All RMSE values are non-negative finite numbers
- For n=100 and all three metabolite groupings, the smallest RMSE is achieved by the Nyholt method under DisCo, outperforming other methods by large margins
- RMSE under defined 8-pathway grouping is lower than the ungrouped configuration for comparable cells
- Median aggregation reproduces the reported superiority of the distance-correlation (DisCo P-ad) adjustment over PrsCo-based Meff

## Limitations

- RMSE is computed against median-aggregated Meff, so per-replicate variability is collapsed and outlier replicates are masked
- Conclusions are specific to the simulated metabolomics scenarios (sample size-to-feature ratios, response types, metabolite groupings) assessed in the study
- Reliance on the permutation-based Meff0 gold standard limits applicability when permutation results are unavailable or too costly to generate

## Evidence

- [results] RMSE comparison of DisCo P-ad vs PrsCo P-ad: "Comparing the different methods using DisCo, for n=100 and all three metabolite groupings, the smallest RMSE was achieved by the Nyholt method, outperforming all the other methods by large margins."
- [abstract] DisCo P-ad proposal: "We propose the distance-correlation-based p-value adjustment (DisCo P-ad) as a novel modification that can enhance existing eigen-analysis-based multiple testing correction procedures"
- [abstract] Study scope: "Our study, including varying sample size-to-feature ratios, response types, and metabolite groupings, highlights the superior performance of the distance correlation."
- [readme] Repository codes: "Check out "Codes-Explained" folder to find all the example R and MATLAB codes (for one data sub-sample scenario) used for the simulation."
