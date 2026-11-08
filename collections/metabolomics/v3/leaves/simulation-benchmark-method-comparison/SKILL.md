---
name: simulation-benchmark-method-comparison
description: Use when you have simulated or sub-sampled high-dimensional metabolomics
  p-values plus competing correlation matrices (e.g., Pearson vs distance correlation)
  and need to benchmark eigen-analysis-based Meff estimation methods against a gold
  standard by RMSE.
when_to_use_negative:
- When you only have raw metabolite feature data and no p-values or correlation matrices
  yet — the skill operates downstream of PrsCo/DisCo matrix construction
- When no permutation-based gold-standard Meff0 is available, since RMSE comparison
  against Meff0 is the core evaluation
- When a single real (non-simulated) dataset is the target — this skill requires replicates
  across sample sizes and groupings to aggregate medians
edam_operation: http://edamontology.org/operation_0306
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3306
tools:
- name: R
  role: Implemented example simulation code and Meff/RMSE comparison analysis (R codes
    in Codes-Explained folder)
  repo: https://github.com/KechrisLab/DisCoPad
- name: MATLAB
  role: Implemented example simulation code for one data sub-sample scenario used
    in the study
  repo: https://github.com/KechrisLab/DisCoPad
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.3390/metabo15010028
    title: 'DisCo P-ad: Distance-Correlation-Based p-Value Adjustment Enhances Multiple
      Testing Corrections for Metabolomics'
  source_content_hash: sha256:0c353f4879b1cfc27a3999370513389a67f1b1e63df588f66cce52042e06c353
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/KechrisLab/DisCoPad
  tools:
  - MATLAB
  - R
  edam_operation: http://edamontology.org/operation_0306
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3306
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# simulation-benchmark-method-comparison

## Summary

A simulation-based benchmarking skill for comparing multiple-testing correction variants: it computes the effective number of tests (Meff) from PrsCo (Pearson correlation) and DisCo (distance correlation) matrices across eigen-analysis-based p-value adjustment methods, groupings, and sample sizes, then evaluates each method's median Meff against a permutation-based gold-standard Meff0 via RMSE.

## When to use

Use when you have simulated or sub-sampled high-dimensional metabolomics p-values plus competing correlation matrices (e.g., Pearson vs distance correlation) and need to benchmark eigen-analysis-based Meff estimation methods against a gold standard by RMSE.

## When NOT to use

- When you only have raw metabolite feature data and no p-values or correlation matrices yet — the skill operates downstream of PrsCo/DisCo matrix construction
- When no permutation-based gold-standard Meff0 is available, since RMSE comparison against Meff0 is the core evaluation
- When a single real (non-simulated) dataset is the target — this skill requires replicates across sample sizes and groupings to aggregate medians

## Inputs

- Simulated metabolomics p-value sub-samples
- PrsCo correlation matrices (Pearson's correlation) produced upstream
- DisCo correlation matrices (distance correlation) produced upstream
- Gold-standard permutation-based Meff0 estimates (e.g., from task_003)

## Outputs

- Full RMSE comparison table (meff_rmse_results.csv) with method, matrix type, grouping, and sample-size columns
- Median Meff estimates aggregated per method x matrix x grouping x sample-size cell

## How to apply

Load the simulated metabolomics p-value sub-samples together with the upstream PrsCo (Pearson's correlation) and DisCo (distance correlation) matrices. For each of the five eigen-analysis-based p-value adjustment methods (including Nyholt), compute Meff from the eigen-decomposition of the PrsCo matrix for each metabolite grouping (ungrouped, defined 8-pathway, random) and each of the three sample sizes (e.g., n=100). Repeat the Meff computation using the DisCo matrix, implementing the distance-correlation-based adjustment (DisCo P-ad) as a modification of each method. Aggregate Meff estimates across simulation replicates by taking the median per method x matrix x grouping x sample-size cell, compute RMSE of the median Meff against the gold-standard Meff0, and write the full RMSE comparison table (e.g., meff_rmse_results.csv) preserving method, matrix type, grouping, and sample-size columns for downstream pattern checks such as DisCo yielding the smallest RMSE for Nyholt at n=100.

## Related tools

- **R** (Implemented example simulation code and Meff/RMSE comparison analysis (R codes in Codes-Explained folder)) — https://github.com/KechrisLab/DisCoPad
- **MATLAB** (Implemented example simulation code for one data sub-sample scenario used in the study) — https://github.com/KechrisLab/DisCoPad

## Evaluation signals

- Output table exists and covers all 2 matrices (PrsCo, DisCo) x 5 methods x 3 groupings (ungrouped, defined 8-pathway, random) x 3 sample sizes cells
- All RMSE values are non-negative finite numbers
- Median (not mean) Meff aggregation across simulation replicates within each cell
- Reproduced pattern: for n=100 and all three metabolite groupings under DisCo, the Nyholt method achieves the smallest RMSE, outperforming the other methods by large margins
- Consistency with the reported superiority of the distance correlation across sample-size-to-feature ratios, response types, and metabolite groupings

## Limitations

- The skill depends on upstream PrsCo/DisCo correlation matrices and a separate permutation-based Meff0 gold standard; it cannot produce RMSE comparisons without them
- Simulation evidence covers varying sample size-to-feature ratios, response types, and metabolite groupings — conclusions may not transfer to other designs
- Only one example R and MATLAB code scenario (for one data sub-sample scenario) is provided in the repository's Codes-Explained folder

## Evidence

- [abstract] Our study, including varying sample size-to-feature ratios, response types, and metabolite groupings, highlights the superior performance of the distance correlation.: "Our study, including varying sample size-to-feature ratios, response types, and metabolite groupings, highlights the superior performance of the distance correlation."
- [intro] Check out "Codes-Explained" folder to find all the example R and MATLAB codes: "Check out "Codes-Explained" folder to find all the example R and MATLAB codes"
- [readme] Refer to the "Read-Me.txt" file to understand what each script accomplishes.: "Refer to the "Read-Me.txt" file to understand what each script accomplishes."
- [abstract] We propose the distance-correlation-based p-value adjustment (DisCo P-ad) as a novel modification that can enhance existing eigen-analysis-based multiple testing correction procedures: "We propose the distance-correlation-based p-value adjustment (DisCo P-ad) as a novel modification that can enhance existing eigen-analysis-based multiple testing correction"
