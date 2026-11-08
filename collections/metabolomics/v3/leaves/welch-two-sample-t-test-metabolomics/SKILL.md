---
name: welch-two-sample-t-test-metabolomics
description: Use when raw per-metabolite p-values are needed from a two-group comparison
  (e.g., sex) of ungrouped log-transformed metabolite abundances via Welch's unequal-variance
  t-tests, prior to eigen-analysis-based FWER adjustment.
when_to_use_negative:
- The comparison has more than two levels or requires covariate adjustment — Welch's
  two-sample t-test supports only a single binary grouping (sex in this application)
- P-values will be corrected with generic FDR/BH rather than the eigen-analysis-based
  FWER procedures (PrsCo, Gao, Galwey and their DisCo P-ad modifications) this pipeline
  is designed to feed
- Features are aggregated into metabolite groups or pathways rather than the 761 ungrouped
  metabolite abundance columns used in the real-data application
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0602
tools:
- name: R
  role: Runs the per-stratum Welch two-sample t-tests of sex against the 761 metabolite
    abundances and the eigen-analysis-based p-value adjustment scripts that produce
    ttest_sig_counts.csv
  repo: github.com/KechrisLab/DisCoPad
- name: MATLAB
  role: Provides example simulation codes alongside R in the Codes-Explained folder
    (one data sub-sample scenario)
  repo: github.com/KechrisLab/DisCoPad
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.3390/metabo15010028
    title: 'DisCo P-ad: Distance-Correlation-Based p-Value Adjustment Enhances Multiple
      Testing Corrections for Metabolomics'
  source_content_hash: sha256:23545f7c608f0dbaff57bfb0958c16759d4947af4a2c8c4afb02a22fd3906b03
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MATLAB
  - R
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 4
---

# welch-two-sample-t-test-metabolomics

## Summary

Applies Welch's unequal-variance two-sample t-tests of a binary grouping (sex) against each of 761 ungrouped log-transformed metabolite abundances within COPDGene strata (GOLD 3/4 severe, n=183; GOLD 0 controls, n=448), producing raw per-metabolite p-values that feed eigen-analysis-based FWER adjustment (PrsCo, Gao, Galwey, and their DisCo P-ad modifications) at FWER alpha=0.05.

## When to use

Use when raw per-metabolite p-values are needed from a two-group comparison (e.g., sex) of ungrouped log-transformed metabolite abundances via Welch's unequal-variance t-tests, prior to eigen-analysis-based FWER adjustment.

## When NOT to use

- The comparison has more than two levels or requires covariate adjustment — Welch's two-sample t-test supports only a single binary grouping (sex in this application)
- P-values will be corrected with generic FDR/BH rather than the eigen-analysis-based FWER procedures (PrsCo, Gao, Galwey and their DisCo P-ad modifications) this pipeline is designed to feed
- Features are aggregated into metabolite groups or pathways rather than the 761 ungrouped metabolite abundance columns used in the real-data application

## Inputs

- COPDGene metabolomics dataset from KechrisLab/DisCoPad: 761 log-transformed metabolite abundance columns plus sex and GOLD stage per subject
- Strata definitions: severe COPD subjects (GOLD 3/4, n=183) and control subjects (GOLD 0, n=448)
- Binary two-level grouping variable (sex) used as the test response

## Outputs

- Vector of 761 raw Welch p-values per stratum (one per ungrouped metabolite)
- ttest_sig_counts.csv: per-method significant metabolite counts at FWER alpha=0.05 for PrsCo, Gao, Galwey, and their DisCo P-ad modifications

## How to apply

Load the metabolomics dataset from the KechrisLab/DisCoPad repository (761 log-transformed metabolite abundance columns plus sex and GOLD stage per subject) and subset subjects into the severe stratum (GOLD 3/4, n=183) and the control stratum (GOLD 0, n=448). In R, apply Welch's two-sample t-test with unequal variances (var.equal=FALSE) of each of the 761 metabolite abundances against sex, collecting exactly one raw p-value per metabolite per stratum. Feed the 761 raw p-values into the eigen-analysis-based family-wise error rate procedures under comparison (PrsCo, Gao, Galwey, and their DisCo P-ad modifications) at FWER alpha=0.05. Count significant metabolites per method per stratum and write results to ttest_sig_counts.csv, expecting DisCo-based Meff's for Gao and Galwey to be higher and closer to the gold-standard than PrsCo in the control stratum. The unequal-variance Welch test is used because group variances of metabolite abundances may differ, and its raw p-value vector is the required input for the distance-correlation-based p-value adjustment evaluated in the paper.

## Related tools

- **R** (Runs the per-stratum Welch two-sample t-tests of sex against the 761 metabolite abundances and the eigen-analysis-based p-value adjustment scripts that produce ttest_sig_counts.csv) — github.com/KechrisLab/DisCoPad
- **MATLAB** (Provides example simulation codes alongside R in the Codes-Explained folder (one data sub-sample scenario)) — github.com/KechrisLab/DisCoPad

## Examples

```
Rscript -e 'pv <- sapply(y761, function(y) t.test(y ~ sex, var.equal=FALSE)$p.value)'
```

## Evaluation signals

- Significant counts in ttest_sig_counts.csv equal the Table 3/Table S1 reported values: severe stratum Galwey+DisCo yields 9 fewer significant metabolites than PrsCo; control stratum Gao and Galwey+DisCo each yield 8 fewer than PrsCo at FWER alpha=0.05
- Each stratum produces exactly 761 raw p-values, one per ungrouped metabolite, before adjustment
- Stratum sample sizes replicate the article: severe n=183 (GOLD 3/4) and controls n=448 (GOLD 0)
- In the control stratum, DisCo-based Meff's for Gao and Galwey are higher and closer to the gold-standard than PrsCo-based Meff's
- T-tests are computed with unequal variances (var.equal=FALSE), not pooled-variance t-tests

## Limitations

- R and MATLAB example codes in the DisCoPad repository cover only one data sub-sample scenario of the simulation, not all evaluated settings
- The real-data design fixes the grouping (sex) and stratification (GOLD 3/4 severe vs GOLD 0 controls); other groupings or strata require re-derivation and re-validation
- DisCo-based adjustment identifies fewer significant metabolites than PrsCo (e.g., 8 fewer for Gao and Galwey+DisCo in controls), which can eliminate false positives but may also drop true positives

## Evidence

- [abstract] abstract: "Our study, including varying sample size-to-feature ratios, response types, and metabolite groupings, highlights the superior performance of the distance correlation."
- [intro] intro: "Check out "Codes-Explained" folder to find all the example R and MATLAB codes (for one data sub-sample scenario) used for the simulation"
- [readme] readme: "Repository that contains links to all the codes and data-files used in the article."
