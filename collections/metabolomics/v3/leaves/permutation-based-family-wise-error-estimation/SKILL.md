---
name: permutation-based-family-wise-error-estimation
description: Use when you need a gold-standard family-wise error rate benchmark (Meff0
  = alpha / alpha_f0) to calibrate or validate eigen-analysis or distance-correlation-based
  p-value adjustment procedures for high-dimensional metabolomics association testing.
when_to_use_negative:
- The goal is a fast p-value adjustment on the observed data itself — this skill is
  a computationally heavy gold-standard benchmark (10,000 permutations with full re-testing),
  not a practical correction procedure like DisCo P-ad
- The response is not a continuous outcome or the features are not well approximated
  by a multivariate-Gaussian with an estimable correlation structure
- Only a small sub-sample of the real data is available, so the empirical correlation
  structure of the 761 metabolites cannot be reliably estimated
edam_operation: http://edamontology.org/operation_3664
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0203
tools:
- name: R
  role: Example R code (for one data sub-sample scenario) used for the simulation,
    provided in the Codes-Explained folder of the DisCoPad repository
  repo: github.com/KechrisLab/DisCoPad
- name: MATLAB
  role: Example MATLAB code (for one data sub-sample scenario) used for the simulation,
    provided in the Codes-Explained folder of the DisCoPad repository
  repo: github.com/KechrisLab/DisCoPad
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.3390/metabo15010028
    title: 'DisCo P-ad: Distance-Correlation-Based p-Value Adjustment Enhances Multiple
      Testing Corrections for Metabolomics'
  source_content_hash: sha256:564ee479b408884af3091430e04cdcfe5482180a1bee9f888a4cb370a1afb6ac
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MATLAB
  - R
  edam_operation: http://edamontology.org/operation_3664
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 5
---

# permutation-based-family-wise-error-estimation

## Summary

Estimates the gold-standard effective number of tests (Meff0 = alpha / alpha_f0) by running K=10,000 label permutations with full re-testing of all marginal metabolite-outcome associations per permutation, establishing an empirical family-wise Type-I error benchmark for multiple-testing correction methods in MWAS.

## When to use

Use when you need a gold-standard family-wise error rate benchmark (Meff0 = alpha / alpha_f0) to calibrate or validate eigen-analysis or distance-correlation-based p-value adjustment procedures for high-dimensional metabolomics association testing.

## When NOT to use

- The goal is a fast p-value adjustment on the observed data itself — this skill is a computationally heavy gold-standard benchmark (10,000 permutations with full re-testing), not a practical correction procedure like DisCo P-ad
- The response is not a continuous outcome or the features are not well approximated by a multivariate-Gaussian with an estimable correlation structure
- Only a small sub-sample of the real data is available, so the empirical correlation structure of the 761 metabolites cannot be reliably estimated

## Inputs

- COPDGene ST001443 metabolite abundance matrix (761 metabolites x 1054 subjects) from the DisCoPad repository
- Continuous logit(FEV1/FVC) outcome vector for the COPDGene subjects
- Sample sizes for sub-sampling scenarios (n=100, 500, 1000)
- Number of permutations (K=10,000) and significance level alpha

## Outputs

- Permutation-estimated family-wise Type-I error rate alpha_f0 for each sample-size scenario
- Gold-standard effective number of tests Meff0 = alpha / alpha_f0 per scenario
- Results table (e.g., meff0_gold_standard.csv) matching Table 1, row 1

## How to apply

Load the COPDGene ST001443 metabolite abundance matrix (761 metabolites x 1054 subjects) and the continuous logit(FEV1/FVC) outcome, draw random sub-samples of n=100, 500, and 1000 subjects, and estimate the empirical multivariate correlation structure of the metabolites at each sample size. Simulate multivariate-Gaussian feature sets matching the estimated correlation structure, paired with the continuous response. For each scenario, run K=10,000 label permutations, recomputing the full set of marginal metabolite-outcome association tests per permutation to build the null distribution of the minimum p-value. Derive Meff0 = alpha / alpha_f0, where alpha_f0 is the permutation-estimated family-wise Type-I error rate at level alpha, and assemble the Meff0 values (e.g., Table 1, row 1) into a results table such as meff0_gold_standard.csv. Expect Meff0 to decrease as sample size increases, since larger n better captures the null association between metabolite abundances and the outcome.

## Related tools

- **R** (Example R code (for one data sub-sample scenario) used for the simulation, provided in the Codes-Explained folder of the DisCoPad repository) — github.com/KechrisLab/DisCoPad
- **MATLAB** (Example MATLAB code (for one data sub-sample scenario) used for the simulation, provided in the Codes-Explained folder of the DisCoPad repository) — github.com/KechrisLab/DisCoPad

## Examples

```
Rscript Codes-Explained/permutation_simulation.R --n 100 --K 10000 --alpha 0.05 --out meff0_gold_standard.csv
```

## Evaluation signals

- All three sample-size scenarios (n=100, 500, 1000) execute with the full K=10,000 permutations each
- Reported Meff0 values are finite positive numbers
- Meff0 ordering is consistent with the paper's Table 1 row 1: Meff0 decreases as sample size increases (alpha_f0 increases with n)
- alpha_f0 is estimated from the null distribution of the minimum p-value across all 761 marginal tests per permutation
- Output CSV (meff0_gold_standard.csv) contains exactly one Meff0 row per sample-size scenario

## Limitations

- Computationally expensive: 10,000 permutations each requiring recomputation of all 761 marginal association tests per scenario
- Relies on multivariate-Gaussian simulation matching the empirical correlation structure, so fidelity depends on the sub-sample correlation estimate
- Meff0 depends on sample size; larger n better captures the null association between metabolite abundances and the outcome, so benchmark values are scenario-specific

## Evidence

- [abstract] permutation gold standard: "Our study, including varying sample size-to-feature ratios, response types, and metabolite groupings, highlights the superior performance of the distance correlation."
- [intro] repository access to codes and data: "Repository that contains links to all the codes and data-files used in the article."
- [intro] example sub-sample scenario code: "Check out "Codes-Explained" folder to find all the example R and MATLAB codes (for one data sub-sample scenario) used for the simulation"
- [readme] script documentation: "Refer to the "Read-Me.txt" file to understand what each script accomplishes."
- [intro] codes and data file entry point: "Click on the "link-to-codes-and-data-objects" file to access all codes and related items."
