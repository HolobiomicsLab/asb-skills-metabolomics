---
name: copd-metabolomics-stratification-by-gold-stage
description: Use when reproducing or extending the real-data COPDGene metabolomics
  result comparing DisCo-based vs PrsCo-based effective-number-of-tests adjustment
  for Welch t-tests within severe COPD (GOLD 3/4) and control (GOLD 0) strata.
when_to_use_negative:
- The analysis goal involves simulated data with metabolite groupings or varying sample
  size-to-feature ratios rather than the ungrouped real-data COPDGene reproduction
- Only a single testing correction (e.g. plain Bonferroni) is needed with no comparison
  of eigen-analysis-based effective-number-of-tests methods
- The dataset is not stratifiable by GOLD stage or lacks the sex covariate required
  for the two-sample t-tests
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: R
  role: Implements Welch's two-sample t-tests of sex against the 761 metabolite abundances
    and the eigen-analysis-based p-value adjustments (PrsCo, Gao, Galwey, DisCo P-ad)
    per stratum
  repo: github.com/KechrisLab/DisCoPad
- name: MATLAB
  role: Example code alternative provided in the repository's Codes-Explained folder
    for the simulation scenario
  repo: github.com/KechrisLab/DisCoPad
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.3390/metabo15010028
    title: 'DisCo P-ad: Distance-Correlation-Based p-Value Adjustment Enhances Multiple
      Testing Corrections for Metabolomics'
  source_content_hash: sha256:a90df218230c2982e7469cbcdc7bfa52f9773843964408473ca76caa85aa853f
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MATLAB
  - R
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# copd-metabolomics-stratification-by-gold-stage

## Summary

Reproduces the COPDGene real-data analysis from the DisCo P-ad paper by stratifying subjects by GOLD stage and running Welch's two-sample t-tests of sex against 761 ungrouped metabolite abundances, then comparing eigen-analysis-based p-value adjustments (PrsCo, Gao, Galwey) with and without DisCo P-ad modification at FWER alpha=0.05.

## When to use

Use when reproducing or extending the real-data COPDGene metabolomics result comparing DisCo-based vs PrsCo-based effective-number-of-tests adjustment for Welch t-tests within severe COPD (GOLD 3/4) and control (GOLD 0) strata.

## When NOT to use

- The analysis goal involves simulated data with metabolite groupings or varying sample size-to-feature ratios rather than the ungrouped real-data COPDGene reproduction
- Only a single testing correction (e.g. plain Bonferroni) is needed with no comparison of eigen-analysis-based effective-number-of-tests methods
- The dataset is not stratifiable by GOLD stage or lacks the sex covariate required for the two-sample t-tests

## Inputs

- COPDGene metabolomics dataset with 761 log-transformed metabolite abundances per subject, plus sex and GOLD stage columns (from github.com/KechrisLab/DisCoPad)
- Stratum definitions: severe COPD (GOLD 3/4, n=183) and control (GOLD 0, n=448)
- FWER threshold alpha=0.05

## Outputs

- ttest_sig_counts.csv containing the raw p-value table and per-method significant metabolite counts
- Per-stratum, per-method counts of significant metabolites (PrsCo, Gao, Galwey, and DisCo P-ad modifications)

## How to apply

Load the metabolomics dataset from the DisCoPad repository (github.com/KechrisLab/DisCoPad) containing 761 log-transformed metabolite abundances plus sex and GOLD stage per subject. Subset subjects into the severe stratum (GOLD 3/4, n=183) and the control stratum (GOLD 0, n=448). In R, per stratum, apply Welch's two-sample t-test (unequal variances) of each of the 761 ungrouped metabolite abundances against sex to get a raw p-value per metabolite. Adjust the 761 raw p-values for family-wise error rate with eigen-analysis-based procedures (PrsCo, Gao, Galwey, and their DisCo P-ad modifications) at FWER alpha=0.05. Count significant metabolites per method per stratum and verify DisCo P-ad differences against PrsCo: 9 fewer significant metabolites for Galwey+DisCo in the severe stratum and 8 fewer for Gao and Galwey+DisCo in the control stratum, matching Table 3/Table S1. Write the p-value table and per-method significant counts to ttest_sig_counts.csv.

## Related tools

- **R** (Implements Welch's two-sample t-tests of sex against the 761 metabolite abundances and the eigen-analysis-based p-value adjustments (PrsCo, Gao, Galwey, DisCo P-ad) per stratum) — github.com/KechrisLab/DisCoPad
- **MATLAB** (Example code alternative provided in the repository's Codes-Explained folder for the simulation scenario) — github.com/KechrisLab/DisCoPad

## Examples

```
Rscript ttest_p_adjust.R --data copdgene_metab.csv --stratum GOLD --severe "3,4" --control 0 --n_metab 761 --alpha 0.05 --out ttest_sig_counts.csv
```

## Evaluation signals

- Per-method significant counts in ttest_sig_counts.csv equal Table 3/Table S1 reported counts at FWER alpha=0.05
- Severe stratum (GOLD 3/4, n=183): Galwey+DisCo identifies 9 fewer significant metabolites than PrsCo
- Control stratum (GOLD 0, n=448): Gao and Galwey with DisCo each identify 8 fewer significant metabolites than PrsCo
- Control stratum: DisCo-based Meff values for Gao and Galwey are higher and closer to the gold-standard than PrsCo
- Stratum sizes match the reported subject counts (n=183 severe, n=448 controls)

## Limitations

- The validated counts are specific to the COPDGene real data with ungrouped metabolites; grouped metabolite scenarios and simulations are handled by separate workflows
- Differences in significant counts may eliminate potential false positives but the paper frames this as potential, not confirmed, error reduction
- The repository provides example R and MATLAB codes for only one data sub-sample scenario used in the simulation, so other scenarios may require adapting scripts

## Evidence

- [abstract] DisCo-based p-value adjustment vs PrsCo in effective number of tests: "Our study, including varying sample size-to-feature ratios, response types, and metabolite groupings, highlights the superior performance of the distance correlation."
- [intro] Repository access to codes and data: "Repository that contains links to all the codes and data-files used in the article."
- [readme] Codes-Explained folder contents: "Check out "Codes-Explained" folder to find all the example R and MATLAB codes (for one data sub-sample scenario) used for the simulation. Refer to the "Read-Me.txt" file to understand what each"
