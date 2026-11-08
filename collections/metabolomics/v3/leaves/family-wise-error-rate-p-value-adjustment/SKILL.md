---
name: family-wise-error-rate-p-value-adjustment
description: Use when a metabolomics or GWAS-style analysis yields many correlated
  raw p-values (e.g., per-metabolite t-tests) and you must control FWER at a fixed
  alpha (e.g., 0.05) while comparing eigen-analysis-based effective-number-of-tests
  estimators.
when_to_use_negative:
- Features are independent or uncorrelated so eigen-analysis-based Meff estimation
  is unnecessary (plain Bonferroni suffices)
- The goal is FDR (expected false discovery proportion) control rather than family-wise
  error rate control
- The test statistics are not per-feature univariate tests with a well-defined correlation
  structure across features
edam_operation: http://edamontology.org/operation_0306
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_2269
tools:
- name: R
  role: Implemented the Welch two-sample t-tests of sex vs. 761 metabolite abundances
    and the eigen-analysis-based FWER p-value adjustments (PrsCo, Gao, Galwey, and
    DisCo P-ad variants); example R scripts are provided in the repository
  repo: github.com/KechrisLab/DisCoPad
- name: MATLAB
  role: Alternative language for the example simulation scripts provided alongside
    R in the Codes-Explained folder
  repo: github.com/KechrisLab/DisCoPad
- name: DisCoPad
  role: Repository containing links to all codes and data-files (metabolomics dataset,
    link-to-codes-and-data-objects file) needed to reproduce the p-value adjustment
    workflow
  repo: github.com/KechrisLab/DisCoPad
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.3390/metabo15010028
    title: 'DisCo P-ad: Distance-Correlation-Based p-Value Adjustment Enhances Multiple
      Testing Corrections for Metabolomics'
  source_content_hash: sha256:accd073cb292c5026e755bd0f4e74e8650a778a177bc77f3c9ff4f43cadbcd99
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - DisCoPad
  - MATLAB
  - R
  edam_operation: http://edamontology.org/operation_0306
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# family-wise-error-rate-p-value-adjustment

## Summary

Adjust raw p-values from high-dimensional metabolomics tests (e.g., 761 Welch two-sample t-tests of sex vs. metabolite abundances) for family-wise error rate using eigen-analysis-based procedures (PrsCo, Gao, Galwey) enhanced with the distance-correlation-based DisCo P-ad modification from the DisCoPad repository.

## When to use

Use when a metabolomics or GWAS-style analysis yields many correlated raw p-values (e.g., per-metabolite t-tests) and you must control FWER at a fixed alpha (e.g., 0.05) while comparing eigen-analysis-based effective-number-of-tests estimators.

## When NOT to use

- Features are independent or uncorrelated so eigen-analysis-based Meff estimation is unnecessary (plain Bonferroni suffices)
- The goal is FDR (expected false discovery proportion) control rather than family-wise error rate control
- The test statistics are not per-feature univariate tests with a well-defined correlation structure across features

## Inputs

- Raw per-feature p-values (e.g., 761 Welch two-sample t-test p-values of sex vs. log-transformed metabolite abundances per stratum)
- Feature-by-feature Pearson's correlation matrix of the tested variables
- Stratified subject-level metabolomics data with covariates (sex, GOLD stage) from the DisCoPad repository

## Outputs

- FWER-adjusted p-value tables under PrsCo, Gao, Galwey, and their DisCo P-ad variants
- Effective number of tests (Meff) estimates per method per stratum
- Per-method significant feature counts at FWER alpha=0.05 (e.g., written to ttest_sig_counts.csv)

## How to apply

In R, compute raw p-values per feature (e.g., Welch's two-sample t-test of sex against each of 761 log-transformed ungrouped metabolite abundances within strata such as GOLD 3/4 severe COPD, n=183, and GOLD 0 controls, n=448). Build the Pearson's correlation matrix of the p-values/features and estimate the effective number of tests (Meff) via eigen-analysis using PrsCo, Gao, and Galwey, then re-apply each with the DisCo P-ad modification, which replaces the Pearson correlation with distance correlation and can raise Meff toward the gold standard, reducing false positives. Adjust the 761 p-values for FWER at alpha=0.05 under each method and count significant metabolites per method per stratum. Codes and data are accessed via the 'link-to-codes-and-data-objects' file in the KechrisLab/DisCoPad repository, with the Read-Me.txt explaining each script.

## Related tools

- **R** (Implemented the Welch two-sample t-tests of sex vs. 761 metabolite abundances and the eigen-analysis-based FWER p-value adjustments (PrsCo, Gao, Galwey, and DisCo P-ad variants); example R scripts are provided in the repository) — github.com/KechrisLab/DisCoPad
- **MATLAB** (Alternative language for the example simulation scripts provided alongside R in the Codes-Explained folder) — github.com/KechrisLab/DisCoPad
- **DisCoPad** (Repository containing links to all codes and data-files (metabolomics dataset, link-to-codes-and-data-objects file) needed to reproduce the p-value adjustment workflow) — github.com/KechrisLab/DisCoPad

## Examples

```
Rscript ttest_fwer_adjust.R --data copd_metabolites.csv --strata GOLD --tests welch --features 761 --alpha 0.05 --methods PrsCo,Gao,Galwey,DisCoPad --out ttest_sig_counts.csv
```

## Evaluation signals

- Per-method significant counts in ttest_sig_counts.csv must match Table 3/Table S1: severe stratum PrsCo baseline with Galwey+DisCo yielding 9 fewer significant metabolites; control stratum with Gao and Galwey+DisCo each 8 fewer, at FWER alpha=0.05
- DisCo-based Meff estimates for Gao and Galwey in the control stratum (n=448, ungrouped metabolites) should be higher and closer to the gold-standard Meff than PrsCo-based estimates
- Exactly 761 raw p-values must be tested per stratum (severe n=183, control n=448) before adjustment
- Adjusted p-values must be monotonically non-decreasing relative to raw p-values, and all counts must be reproducible from the DisCoPad scripts

## Limitations

- Performance depends on sample size-to-feature ratios, response types, and metabolite groupings; benefits were demonstrated via simulations varying these factors rather than guaranteed universally
- Fewer significant metabolites under DisCo P-ad (e.g., 8–9 fewer than PrsCo) may eliminate false positives but could also drop true positives; it is a trade-off in power vs. false positives
- Reductions in significant counts were stratum- and method-specific (e.g., no reported reduction for Gao in the severe stratum), so effects do not generalize across all methods and strata

## Evidence

- [abstract] We propose the distance-correlation-based p-value adjustment (DisCo P-ad) as a novel modification that can enhance existing eigen-analysis-based multiple testing correction procedures by increasing: "We propose the distance-correlation-based p-value adjustment (DisCo P-ad) as a novel modification that can enhance existing eigen-analysis-based multiple testing correction procedures by increasing"
- [abstract] Our study, including varying sample size-to-feature ratios, response types, and metabolite groupings, highlights the superior performance of the distance correlation.: "Our study, including varying sample size-to-feature ratios, response types, and metabolite groupings, highlights the superior performance of the distance correlation."
- [readme] Click on the "link-to-codes-and-data-objects" file to access all codes and related items.: "Click on the "link-to-codes-and-data-objects" file to access all codes and related items."
- [readme] Check out "Codes-Explained" folder to find all the example R and MATLAB codes (for one data sub-sample scenario) used for the simulation.: "Check out "Codes-Explained" folder to find all the example R and MATLAB codes (for one data sub-sample scenario) used for the simulation."
