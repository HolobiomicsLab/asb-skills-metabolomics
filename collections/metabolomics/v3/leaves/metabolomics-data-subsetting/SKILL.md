---
name: metabolomics-data-subsetting
description: Use when you need to assess how sample size affects the estimated correlation
  structure of high-dimensional metabolite abundances, or to build simulated null
  scenarios for validating multiple-testing corrections like Meff0 = alpha / alpha_f0.
when_to_use_negative:
- Your goal is to compute DisCo P-ad or eigen-analysis-based p-value adjustments on
  the full cohort rather than building permutation-based gold-standard Meff0 benchmarks
  from simulated subsets
- The input is not a subject-level metabolite abundance matrix paired with a continuous
  outcome (e.g. it is already a correlation matrix with no subject identifiers to
  sub-sample)
edam_operation: http://edamontology.org/operation_2239
edam_topics:
- http://edamontology.org/topic_3053
tools:
- name: R
  role: Used to load the metabolite matrix, draw random sub-samples, estimate correlation
    structures, and run permutation simulations (example codes provided in the Codes-Explained
    folder)
  repo: github.com/KechrisLab/DisCoPad
- name: MATLAB
  role: Alternative implementation language for the one data sub-sample scenario simulation
    codes provided in the Codes-Explained folder
  repo: github.com/KechrisLab/DisCoPad
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.3390/metabo15010028
    title: 'DisCo P-ad: Distance-Correlation-Based p-Value Adjustment Enhances Multiple
      Testing Corrections for Metabolomics'
  source_content_hash: sha256:f92440908b4c839ed4989b8ab96ca9d47c4f196c2e008a0cdea51442c7524a95
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MATLAB
  - R
  edam_operation: http://edamontology.org/operation_2239
  edam_topics:
  - http://edamontology.org/topic_3053
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# metabolomics-data-subsetting

## Summary

Draws random sub-samples (n=100, 500, 1000) of subjects from the COPDGene ST001443 metabolite abundance matrix (761 metabolites x 1054 subjects) to estimate empirical multivariate correlation structures, which are then used to simulate multivariate-Gaussian feature sets for permutation-based effective-number-of-tests (Meff0) benchmarking.

## When to use

Use when you need to assess how sample size affects the estimated correlation structure of high-dimensional metabolite abundances, or to build simulated null scenarios for validating multiple-testing corrections like Meff0 = alpha / alpha_f0.

## When NOT to use

- Your goal is to compute DisCo P-ad or eigen-analysis-based p-value adjustments on the full cohort rather than building permutation-based gold-standard Meff0 benchmarks from simulated subsets
- The input is not a subject-level metabolite abundance matrix paired with a continuous outcome (e.g. it is already a correlation matrix with no subject identifiers to sub-sample)

## Inputs

- COPDGene ST001443 metabolite abundance matrix (761 metabolites x 1054 subjects) from the DisCoPad repository data links
- Continuous logit(FEV1/FVC) outcome per subject
- Target sub-sample sizes (n=100, n=500, n=1000)
- Number of permutations K=10,000 and significance level alpha

## Outputs

- Estimated empirical multivariate correlation structure of the 761 metabolites per sub-sample size
- Per-scenario Meff0 = alpha / alpha_f0 values for n=100, 500, 1000
- meff0_gold_standard.csv results table corresponding to Table 1, row 1

## How to apply

Load the COPDGene ST001443 metabolite abundance matrix (761 metabolites x 1054 subjects) and the continuous logit(FEV1/FVC) outcome from the files linked via the DisCoPad repository's 'link-to-codes-and-data-objects' file. Draw random sub-samples of n=100, n=500, and n=1000 subjects and estimate the empirical multivariate Pearson correlation structure of the 761 metabolites for each sample size. Simulate multivariate-Gaussian feature sets matching each estimated correlation structure, paired with the continuous logit(FEV1/FVC) response. Run K=10,000 label permutations per scenario, recomputing all marginal metabolite-outcome association tests per permutation to build the null distribution of the minimum p-value, then derive Meff0 = alpha / alpha_f0. The rationale is that larger n better captures the null association structure, so Meff0 is expected to decrease as n increases (Table 1, row 1). The repository's 'Codes-Explained' folder provides example R and MATLAB codes for one data sub-sample scenario, and its 'Read-Me.txt' explains what each script accomplishes.

## Related tools

- **R** (Used to load the metabolite matrix, draw random sub-samples, estimate correlation structures, and run permutation simulations (example codes provided in the Codes-Explained folder)) — github.com/KechrisLab/DisCoPad
- **MATLAB** (Alternative implementation language for the one data sub-sample scenario simulation codes provided in the Codes-Explained folder) — github.com/KechrisLab/DisCoPad

## Examples

```
idx <- sample(seq_len(1054), 500); sub <- metab[idx, ]; Sigma <- cor(sub)
```

## Evaluation signals

- The script executes all three sample-size scenarios (n=100, 500, 1000) with 10,000 permutations each
- Reported Meff0 values are finite positive numbers
- The three Meff0 values decrease monotonically as sample size increases (alpha_f0 increases), consistent in ordering with Table 1 row 1 for the COPDGene data scenario
- Output is assembled into a single meff0_gold_standard.csv matching the Table 1, row 1 schema

## Limitations

- Meff0 decreased with sample size because larger n better captured the null association between metabolite abundances and the outcome; this trend is data/scenario specific and should not be assumed universal
- Only one data sub-sample scenario has example R and MATLAB codes provided in the repository; other scenarios must be adapted by the user
- Simulation assumes multivariate-Gaussian features matching the empirical correlation structure, which may not hold for non-Gaussian metabolomics data

## Evidence

- [results] The permutation-based gold-standard Meff0 decreased as sample size increased (with alpha_f0 increasing), most likely because a larger n better captured the null association between metabolite abundances and the outcome: [evidence span withheld — anchor status: not_found]
- [intro] Repository provides links to all codes and data-files: "Repository that contains links to all the codes and data-files used in the article."
- [readme] Codes-Explained folder contents: "Check out "Codes-Explained" folder to find all the example R and MATLAB codes (for one data sub-sample scenario) used for the simulation."
- [abstract] Simulation study design: "Our study, including varying sample size-to-feature ratios, response types, and metabolite groupings, highlights the superior performance of the distance correlation."
