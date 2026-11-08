---
name: reported-table-count-reproduction
description: Use when a paper reports exact per-method counts of significant features
  (e.g., significant metabolites at FWER alpha=0.05 in Table 3/Table S1) and you must
  verify those counts by re-executing the analysis from the released data and code.
when_to_use_negative:
- The target result is a continuous statistic, effect size, or figure rather than
  an exact integer count of significant features that can be matched to a table
- The analysis uses grouped metabolites or a different test than Welch's two-sample
  t-test, since the reported counts being reproduced apply to 761 ungrouped metabolites
  tested against sex
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0091
tools:
- name: R
  role: Used to run Welch's two-sample t-tests and the eigen-analysis-based FWER p-value
    adjustment procedures (PrsCo, Gao, Galwey, DisCo P-ad) and to write ttest_sig_counts.csv
  repo: github.com/KechrisLab/DisCoPad
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.3390/metabo15010028
    title: 'DisCo P-ad: Distance-Correlation-Based p-Value Adjustment Enhances Multiple
      Testing Corrections for Metabolomics'
  source_content_hash: sha256:115ecbba77980e3364054f1c9c84274b1eabe75a500444f6de2fbeea86376a2f
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# reported-table-count-reproduction

## Summary

Reproduces published significance-count tables (e.g., Table 3/Table S1) by re-running the paper's statistical pipeline — here Welch's two-sample t-tests of sex against 761 log-transformed metabolite abundances with eigen-analysis-based FWER corrections (PrsCo, Gao, Galwey, and DisCo P-ad variants) — and validating that per-method significant metabolite counts match the reported numbers exactly.

## When to use

Use when a paper reports exact per-method counts of significant features (e.g., significant metabolites at FWER alpha=0.05 in Table 3/Table S1) and you must verify those counts by re-executing the analysis from the released data and code.

## When NOT to use

- The target result is a continuous statistic, effect size, or figure rather than an exact integer count of significant features that can be matched to a table
- The analysis uses grouped metabolites or a different test than Welch's two-sample t-test, since the reported counts being reproduced apply to 761 ungrouped metabolites tested against sex

## Inputs

- Metabolomics dataset from KechrisLab/DisCoPad: 761 log-transformed metabolite abundances per subject, plus sex and GOLD stage columns
- Subject strata definitions: severe COPD (GOLD 3/4, n=183) and control (GOLD 0, n=448)
- Published Table 3 / Table S1 significant-count values used as the ground truth

## Outputs

- ttest_sig_counts.csv containing per-metabolite raw p-values and per-method significant counts
- Per-method significant metabolite counts for PrsCo, Gao, Galwey, and DisCo P-ad variants in each stratum at FWER alpha=0.05

## How to apply

Load the metabolomics dataset from the DisCoPad repository (github.com/KechrisLab/DisCoPad) with 761 log-transformed metabolite abundances plus sex and GOLD stage per subject, and subset subjects into strata defined by the table being reproduced (severe COPD, GOLD 3/4, n=183; controls, GOLD 0, n=448). In R, apply Welch's two-sample t-test (unequal variances) of sex against each of the 761 ungrouped metabolite abundances to obtain one raw p-value per metabolite per stratum. Adjust the 761 raw p-values for family-wise error rate using the eigen-analysis-based procedures under comparison — PrsCo, Gao, Galwey, and their DisCo P-ad modifications — at FWER alpha=0.05, then count significant metabolites per method per stratum. Reproduction succeeds only when the counts equal the published values: severe stratum, PrsCo baseline with Galwey+DisCo 9 fewer significant metabolites; control stratum, PrsCo baseline with Gao and Galwey+DisCo each 8 fewer. Write the p-value table and per-method significant counts to ttest_sig_counts.csv as the checkable artifact.

## Related tools

- **R** (Used to run Welch's two-sample t-tests and the eigen-analysis-based FWER p-value adjustment procedures (PrsCo, Gao, Galwey, DisCo P-ad) and to write ttest_sig_counts.csv) — github.com/KechrisLab/DisCoPad

## Examples

```
Rscript run_ttests_and_adjust.R --data DisCoPad/metabolites.csv --strata GOLD=="3/4",GOLD==0 --methods PrsCo,Gao,Galwey,DisCoPad --alpha 0.05 --out ttest_sig_counts.csv
```

## Evaluation signals

- Significant counts in ttest_sig_counts.csv equal Table 3/Table S1: severe stratum shows Galwey+DisCo 9 fewer significant metabolites than PrsCo at FWER alpha=0.05
- Control stratum (n=448) shows Gao+DisCo and Galwey+DisCo each 8 fewer significant metabolites than PrsCo
- DisCo-based Meff estimates (Gao and Galwey methods) are higher and closer to the gold-standard Meff than PrsCo-based estimates in the control stratum
- Each stratum contains the expected number of subjects (severe n=183, controls n=448) and 761 ungrouped metabolite p-values are produced per method
- All adjustment methods are run at the same FWER threshold alpha=0.05 as the published table

## Limitations

- Counts are specific to the ungrouped 761-metabolite configuration and Welch's t-test against sex; other groupings, response types, or sample size-to-feature ratios may yield different counts
- Reproduction depends on the exact dataset version and strata definitions released in the DisCoPad repository; discrepancies may reflect data updates rather than method errors

## Evidence

- [results] In the COPD control stratum (n=448, ungrouped metabolites), the DisCo-based Meff's for the Gao and Galwey methods were higher and closer to the gold-standard than PrsCo: [evidence span withheld — anchor status: not_found]
- [methods] Adjust the 761 raw p-values for family-wise error rate with the eigen-analysis-based procedures under comparison (PrsCo, Gao, Galwey, and their DisCo P-ad modifications) at FWER alpha=0.05: [evidence span withheld — anchor status: not_found]
- [readme] Repository that contains links to all the codes and data-files used in the article.: "Repository that contains links to all the codes and data-files used in the article."
- [abstract] We propose the distance-correlation-based p-value adjustment (DisCo P-ad) as a novel modification that can enhance existing eigen-analysis-based multiple testing correction procedures: "We propose the distance-correlation-based p-value adjustment (DisCo P-ad) as a novel modification that can enhance existing eigen-analysis-based multiple testing correction procedures"
