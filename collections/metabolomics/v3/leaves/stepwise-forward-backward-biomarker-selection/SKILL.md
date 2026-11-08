---
name: stepwise-forward-backward-biomarker-selection
description: Use when many candidate biomarkers are individually associated with survival
  but are highly correlated, and you must select a reduced subset that remains independently
  significant after mutual adjustment.
when_to_use_negative:
- When the goal is univariate ranking of all biomarkers rather than a mutually adjusted
  subset — run the basic per-cohort Cox meta-analysis across all 226 biomarkers instead.
- When biomarkers are weakly correlated and all remain significant after mutual adjustment
  — stepwise selection is unnecessary.
- When no per-cohort survival summary statistics or individual-level covariate-adjusted
  Cox models are available; the procedure requires successive rounds of fully adjusted
  meta-analyses.
tools:
- name: R
  role: Scripting environment in which the forward-backward selection loop, per-cohort
    Cox fits, and Bonferroni threshold checks are implemented
- name: R-package meta
  role: Fixed-effect meta-analysis of cohort-level Cox estimates at each forward/backward
    round of the stepwise procedure
- name: STATA/SE 11.2
  role: Alternative environment used in the study for the individual-cohort survival
    analyses feeding the meta-analyses
provenance:
  source_task_ids:
  - task_001
  - task_005
  source_papers:
  - doi: 10.1038/s41467-019-11311-9
    title: A metabolic profile of all-cause mortality risk identified in an observational
      study of 44,168 individuals
  source_content_hash: sha256:186d4201c344f13e082f49f3f7e1bf8332f6702a81bba6930c05ab6b20c798c8
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - R-package meta
  - STATA/SE 11.2
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# stepwise-forward-backward-biomarker-selection

## Summary

A multivariate biomarker selection procedure that identifies the subset of highly correlated metabolic biomarkers independently associated with a time-to-event outcome, by iterating forward-backward selection over successive rounds of fixed-effect Cox survival meta-analyses in the R-package meta.

## When to use

Use when many candidate biomarkers are individually associated with survival but are highly correlated, and you must select a reduced subset that remains independently significant after mutual adjustment.

## When NOT to use

- When the goal is univariate ranking of all biomarkers rather than a mutually adjusted subset — run the basic per-cohort Cox meta-analysis across all 226 biomarkers instead.
- When biomarkers are weakly correlated and all remain significant after mutual adjustment — stepwise selection is unnecessary.
- When no per-cohort survival summary statistics or individual-level covariate-adjusted Cox models are available; the procedure requires successive rounds of fully adjusted meta-analyses.

## Inputs

- Per-cohort Cox proportional hazards summary statistics for 226 metabolic biomarkers (e.g., BBMRI-omics atlas deposits), with ln(HR), standard error, cohort ID
- Prespecified 63-biomarker candidate subset used for selection (to prevent overfitting)
- Cohort-level preprocessed biomarker values: x+1 for biomarkers containing zeroes, log-transformed, scaled to standard deviation units separately per study

## Outputs

- Table of the 14 independently associated biomarkers with HR, 95% CI, P value, I² heterogeneity statistic, and P(het) from the fixed-effect meta-analysis (e.g., table2_biomarker_meta_analysis.tsv / stepwise_selected_biomarkers.csv)
- Direction-of-effect annotation per biomarker (lower risk vs higher risk for mortality)

## How to apply

Do NOT run selection on all 226 biomarkers; restrict candidates to a prespecified subset of 63 biomarkers to prevent overfitting. Load per-cohort Cox proportional hazards results (age at blood sampling as time scale, adjusted for sex and study-specific covariates), and initialize the working model with the single biomarker having the lowest P value in the basic-model fixed-effect meta-analysis. In each forward step, add the unselected biomarker with the lowest P value in the current fully adjusted meta-analysis; in each backward step, drop any biomarker whose P value rose above the Bonferroni threshold of 2.21 × 10⁻⁴ (0.05/226). Repeat until every unselected candidate exceeds the threshold, then pool the retained biomarkers via fixed-effect meta-analysis in R-package meta. In the source study this converged to 14 biomarkers (citrate dropped out due to limited additional contribution): lower-risk markers XXL-VLDL-L (HR 0.80, 95% CI 0.75–0.85), S-HDL-L, VLDL-D, PUFA/FA, His, Leu, Val, Alb; higher-risk markers Glc (HR 1.16, 95% CI 1.13–1.19), Lac, Ile, Phe, AcAce, GlycA (HR 1.32, 95% CI 1.27–1.38).

## Related tools

- **R** (Scripting environment in which the forward-backward selection loop, per-cohort Cox fits, and Bonferroni threshold checks are implemented)
- **R-package meta** (Fixed-effect meta-analysis of cohort-level Cox estimates at each forward/backward round of the stepwise procedure)
- **STATA/SE 11.2** (Alternative environment used in the study for the individual-cohort survival analyses feeding the meta-analyses)

## Examples

```
library(meta); metagen(TE = log(HR), seTE = se_logHR, sm = "HR", method = "fixed")  # per-round fixed-effect pooling in the forward-backward loop
```

## Evaluation signals

- The procedure converges to exactly 14 independently associated biomarkers from the 63-biomarker candidate subset.
- Tabulated HRs and 95% CIs match published Table 2 values, e.g., GlycA HR 1.32 (95% CI 1.27–1.38) and XXL-VLDL-L HR 0.80 (95% CI 0.75–0.85).
- Bonferroni threshold applied is exactly 2.21 × 10⁻⁴ = 0.05/226, not 0.05/63.
- Removal checks are symmetric: after each forward step, any retained biomarker whose P value rose above threshold is removed before the next round.
- Heterogeneity statistics (I², P(het)) are reported for each retained biomarker to confirm the fixed-effect pooling is defensible.

## Limitations

- Selection was restricted to 63 of 226 biomarkers to reduce overfitting risk, so biomarkers outside this prespecified subset are never selected even if significant.
- Citrate dropped out during the procedure due to limited additional contribution — correlated markers may be discarded for incremental-value reasons, not lack of association.
- The resulting metabolic biomarker score is not yet suitable for clinical patient classification because it is based on per-cohort scaled biomarker values; concentration-unit scores require individual-level data.
- The forward-backward result depends on the basic-model meta-analysis ranking used for initialization.

## Evidence

- [abstract] We apply a stepwise (forward-backward) procedure based on meta-analysis results and identify 14 circulating biomarkers independently associating with all-cause mortality.: "We apply a stepwise (forward-backward) procedure based on meta-analysis results and identify 14 circulating biomarkers independently associating with all-cause mortality."
- [discussion] the selection of independently associated biomarkers was based on a subset of 63 biomarkers to prevent overfitting: "the selection of independently associated biomarkers was based on a subset of 63 biomarkers to prevent overfitting"
- [discussion] Biomarkers were considered significant when the P value was below the Bonferroni-adjusted threshold of 2.21 × 10−4 (0.05/226): "Biomarkers were considered significant when the P value was below the Bonferroni-adjusted threshold of 2.21 × 10−4 (0.05/226)"
- [discussion] the meta-analyses were performed using a fixed-effect model implemented in the R-package meta: "the meta-analyses were performed using a fixed-effect model implemented in the R-package meta"
- [discussion] we performed a stepwise (forward-backward) procedure based on successive rounds of meta-analyses: "we performed a stepwise (forward-backward) procedure based on successive rounds of meta-analyses"
