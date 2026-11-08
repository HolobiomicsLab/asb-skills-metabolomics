---
name: nmr-metabolomics-biomarker-annotation
description: Use when many mutually correlated candidate biomarkers (e.g. 226 NMR
  metabolic measures) must be pruned to a subset independently associated with a survival
  endpoint, using repeated meta-analytic forward-backward selection.
when_to_use_negative:
- The goal is building a weighted predictive risk score (weights = ln(hazard ratio)
  trained on the Estonian Biobank cohort) rather than selecting independently associated
  biomarkers.
- Input is raw NMR spectral data needing biomarker quantification — this skill assumes
  the 226 Nightingale-platform biomarker values are already measured, log-transformed,
  and scaled per study.
- The full set of 226 biomarkers must be ranked by univariate association; the selection
  step is deliberately restricted to the prespecified 63-biomarker subset.
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_3053
- http://edamontology.org/topic_3517
- http://edamontology.org/topic_0091
tools:
- name: R
  role: Runs the per-cohort Cox proportional hazards models and the stepwise forward-backward
    selection loop
- name: R-package meta
  role: Performs the fixed-effect meta-analysis of cohort-specific biomarker-mortality
    associations at each selection round
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-019-11311-9
    title: A metabolic profile of all-cause mortality risk identified in an observational
      study of 44,168 individuals
  source_content_hash: sha256:217bf377081bb0c7d5f0f198e579cb8c69b78dfbdf7bb69575d30422347d1fd4
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - R-package meta
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_3053
  - http://edamontology.org/topic_3517
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# nmr-metabolomics-biomarker-annotation

## Summary

Reconstructs and applies the study's stepwise (forward-backward) selection procedure that reduces a pool of 63 correlated, mortality-associated NMR metabolic biomarkers to the 14 independently associated with all-cause mortality, using successive rounds of fixed-effect Cox meta-analysis and a Bonferroni threshold of 2.21 × 10⁻⁴.

## When to use

Use when many mutually correlated candidate biomarkers (e.g. 226 NMR metabolic measures) must be pruned to a subset independently associated with a survival endpoint, using repeated meta-analytic forward-backward selection.

## When NOT to use

- The goal is building a weighted predictive risk score (weights = ln(hazard ratio) trained on the Estonian Biobank cohort) rather than selecting independently associated biomarkers.
- Input is raw NMR spectral data needing biomarker quantification — this skill assumes the 226 Nightingale-platform biomarker values are already measured, log-transformed, and scaled per study.
- The full set of 226 biomarkers must be ranked by univariate association; the selection step is deliberately restricted to the prespecified 63-biomarker subset.

## Inputs

- 63-biomarker subset of mortality-associated candidates (from 226 Nightingale NMR metabolic biomarkers)
- Per-cohort Cox proportional hazards results with age at blood sampling as time scale, adjusted for age, sex, and study-specific covariates
- Fully adjusted meta-analysis P values computed per selection round

## Outputs

- Set of 14 independently associated biomarkers with pooled hazard ratios and 95% CIs matching Table 2 (e.g. GlycA HR = 1.32, 95% CI 1.27–1.38; XXL-VLDL-L HR = 0.80, 95% CI 0.75–0.85)
- stepwise_selected_biomarkers.csv

## How to apply

Load the prespecified 63-biomarker subset of mortality-associated candidates (drawn from the 226 Nightingale NMR biomarkers) together with per-cohort Cox results computed with age at blood sampling as the time scale, adjusted for age, sex, and study-specific covariates, into R. Initialize the working model with the single biomarker having the lowest P value in the fixed-effect meta-analysis (R-package meta) of the basic model. In each forward step, add the unselected biomarker with the lowest P value from that round's fully adjusted meta-analysis; after each addition, perform a backward step removing any biomarker whose P value rose above the Bonferroni-adjusted threshold of 2.21 × 10⁻⁴ (0.05/226). Repeat until every biomarker in the working model is below the threshold and every unselected candidate is above it. The rationale for restricting candidates to 63 biomarkers is to prevent overfitting, and the forward-backward structure re-tests earlier entries so correlated biomarkers that lose independence after later additions are removed.

## Related tools

- **R** (Runs the per-cohort Cox proportional hazards models and the stepwise forward-backward selection loop)
- **R-package meta** (Performs the fixed-effect meta-analysis of cohort-specific biomarker-mortality associations at each selection round)

## Examples

```
m <- metagen(TE = lnHR, seTE = se_lnHR, studlab = cohort, sm = "HR", method = "inverse")  # fixed-effect meta-analysis per stepwise round
```

## Evaluation signals

- The procedure converges to exactly 14 independently associated biomarkers.
- All retained biomarkers have P values below the Bonferroni-adjusted threshold of 2.21 × 10⁻⁴ (0.05/226) in the final fully adjusted meta-analysis.
- Pooled HRs and 95% CIs match Table 2, e.g. GlycA HR = 1.32 (1.27–1.38) and XXL-VLDL-L HR = 0.80 (0.75–0.85).
- The selection pool contains only the prespecified 63 biomarkers, never all 226.
- Output file stepwise_selected_biomarkers.csv contains the 14 biomarkers with their pooled statistics.

## Limitations

- Selection was restricted to 63 of 226 biomarkers to prevent overfitting; biomarkers outside this subset were never tested for independent association.
- The targeted NMR platform captures only a fraction of metabolites in human serum, so the resulting biomarker set is platform-limited.
- The final score built from these biomarkers is based on scaled biomarker values created separately per cohort and is not yet suitable for clinical patient classification.

## Evidence

- [abstract] We apply a stepwise (forward-backward) procedure based on meta-analysis results and identify 14 circulating biomarkers independently associating with all-cause mortality.: "We apply a stepwise (forward-backward) procedure based on meta-analysis results and identify 14 circulating biomarkers independently associating with all-cause mortality."
- [discussion] Selection pool restriction: "the selection of independently associated biomarkers was based on a subset of 63 biomarkers to prevent overfitting"
- [discussion] Bonferroni threshold: "Biomarkers were considered significant when the P value was below the Bonferroni-adjusted threshold of 2.21 × 10−4 (0.05/226)"
- [discussion] Meta-analysis tool: "the meta-analyses were performed using a fixed-effect model implemented in the R-package meta"
- [discussion] Stepwise procedure detail: "we performed a stepwise (forward-backward) procedure based on successive rounds of meta-analyses"
