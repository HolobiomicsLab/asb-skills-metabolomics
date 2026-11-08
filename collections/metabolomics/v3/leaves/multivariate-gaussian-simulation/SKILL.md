---
name: multivariate-gaussian-simulation
description: Use when you need controlled simulated data that preserves the empirical
  multivariate correlation structure of a real metabolite abundance matrix (e.g.,
  to compute permutation-based effective number of tests, Meff0, across sample sizes
  n=100/500/1000).
when_to_use_negative:
- When the analysis goal is to evaluate a correction on real (observed) data rather
  than on controlled simulated null scenarios
- When the outcome is binary/categorical and the correlation-structure-preserving
  Gaussian simulation design from the paper (continuous logit(FEV1/FVC) response)
  is not adapted appropriately
edam_operation: http://edamontology.org/operation_3557
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_2269
tools:
- name: R
  role: Used for the example simulation codes (one data sub-sample scenario) in the
    Codes-Explained folder
  repo: github.com/KechrisLab/DisCoPad
- name: MATLAB
  role: Used for the example simulation codes (one data sub-sample scenario) in the
    Codes-Explained folder
  repo: github.com/KechrisLab/DisCoPad
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.3390/metabo15010028
    title: 'DisCo P-ad: Distance-Correlation-Based p-Value Adjustment Enhances Multiple
      Testing Corrections for Metabolomics'
  source_content_hash: sha256:cb6ff9c73d9613787db993aa45cf902dd854b1576e8591f112214d8a4ea6c390
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MATLAB
  - R
  edam_operation: http://edamontology.org/operation_3557
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# multivariate-gaussian-simulation

## Summary

Simulate multivariate-Gaussian feature sets whose correlation structure matches that estimated from real metabolomics data (e.g., COPDGene ST001443, 761 metabolites x 1054 subjects), enabling controlled simulation studies of multiple-testing correction procedures such as the permutation-based gold-standard Meff0.

## When to use

Use when you need controlled simulated data that preserves the empirical multivariate correlation structure of a real metabolite abundance matrix (e.g., to compute permutation-based effective number of tests, Meff0, across sample sizes n=100/500/1000).

## When NOT to use

- When the analysis goal is to evaluate a correction on real (observed) data rather than on controlled simulated null scenarios
- When the outcome is binary/categorical and the correlation-structure-preserving Gaussian simulation design from the paper (continuous logit(FEV1/FVC) response) is not adapted appropriately

## Inputs

- Metabolite abundance matrix (e.g., COPDGene ST001443: 761 metabolites x 1054 subjects)
- Continuous outcome variable (logit(FEV1/FVC))
- Sample-size scenarios for sub-sampling (n=100, 500, 1000)
- Number of permutations K (K=10,000)
- Family-wise significance level alpha

## Outputs

- Simulated multivariate-Gaussian feature sets matching the empirical correlation structure per sample size
- Permutation-based gold-standard Meff0 = alpha / alpha_f0 values per sample size
- Results table (e.g., meff0_gold_standard.csv) corresponding to Table 1 row 1

## How to apply

First, load the real data — for the DisCoPad simulations this is the COPDGene ST001443 metabolite abundance matrix (761 metabolites x 1054 subjects) with the continuous logit(FEV1/FVC) outcome — and draw random sub-samples of n=100, 500, and 1000 subjects to estimate the empirical correlation structure at each sample size. Then generate multivariate-Gaussian feature sets matching the estimated correlation matrix, paired with the continuous response, using the example R and MATLAB codes provided in the repository's Codes-Explained folder for one data sub-sample scenario. Key decision point: sample size determines how well the estimated correlation structure captures null metabolite-outcome associations, which directly affects alpha_f0 and hence Meff0 = alpha / alpha_f0. For each simulated scenario, run K=10,000 label permutations, recomputing the full set of marginal association tests per permutation to build the null distribution of the minimum p-value. Assemble the resulting Meff0 values per sample size into a results table (e.g., meff0_gold_standard.csv) corresponding to Table 1.

## Related tools

- **R** (Used for the example simulation codes (one data sub-sample scenario) in the Codes-Explained folder) — github.com/KechrisLab/DisCoPad
- **MATLAB** (Used for the example simulation codes (one data sub-sample scenario) in the Codes-Explained folder) — github.com/KechrisLab/DisCoPad

## Examples

```
Rscript Codes-Explained/simulation_subsample.R  # run the example R/MATLAB simulation for one data sub-sample scenario from the DisCoPad repository
```

## Evaluation signals

- All three sample-size scenarios (n=100, 500, 1000) execute with the full K=10,000 permutations each
- Reported Meff0 values are finite positive numbers
- Ordering of Meff0 values matches the paper's Table 1 row 1 for the COPDGene scenario (Meff0 decreases as sample size increases, with alpha_f0 increasing)
- Simulated feature sets reproduce the empirical correlation structure estimated from the real COPDGene sub-samples

## Limitations

- The repository provides example R and MATLAB codes for only one data sub-sample scenario; other scenarios require adapting the scripts
- The finding that Meff0 decreases with increasing sample size is a simulation-specific observation, attributed to larger n better capturing null associations between metabolite abundances and the outcome

## Evidence

- [intro] Repository provides links to all codes and data-files used in the article: "Repository that contains links to all the codes and data-files used in the article."
- [intro] Example codes for one sub-sample scenario: "find all the example R and MATLAB codes (for one data sub-sample scenario) used for the simulation"
- [abstract] Simulation scope includes varying sample size-to-feature ratios: "Our study, including varying sample size-to-feature ratios, response types, and metabolite groupings, highlights the superior performance of the distance correlation."
- [readme] Repository navigation instructions: "Check out "Codes-Explained" folder to find all the example R and MATLAB codes (for one data sub-sample scenario) used for the simulation. Refer to the "Read-Me.txt" file"
