---
name: cox-proportional-hazards-survival-analysis
description: Use when cohort-level survival summary statistics (e.g. 226 Nightingale
  NMR metabolic biomarkers vs. all-cause mortality) must be combined across cohorts
  via fixed-effect meta-analysis to select an independent, significant biomarker set.
when_to_use_negative:
- 'The goal is clinical patient classification: the article notes the score built
  from per-cohort scaled biomarker values ''is not yet suitable for classification
  of patients in the clinic''; a concentration-unit score from individual-level data
  is needed instead.'
- Biomarker values are already transformed/scaled — applying x+1, log-transform, and
  per-cohort scaling again would double-transform the data.
- The question is predictive performance benchmarking (C-statistic, IDI against conventional
  risk factors) rather than identification of independently associated biomarkers.
edam_operation: http://edamontology.org/operation_0314
edam_topics:
- http://edamontology.org/topic_2269
- http://edamontology.org/topic_3270
tools:
- name: R
  role: Environment for per-cohort Cox proportional hazards fitting and the stepwise
    forward-backward selection procedure
- name: R-package meta
  role: Fixed-effect meta-analysis combining cohort-level biomarker-mortality hazard
    ratios in each selection round
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-019-11311-9
    title: A metabolic profile of all-cause mortality risk identified in an observational
      study of 44,168 individuals
  source_content_hash: sha256:aabe97814a263489393c0fde0f3b872d24e9e3e5484325f7d51a5fb95069eaa8
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - R-package meta
  edam_operation: http://edamontology.org/operation_0314
  edam_topics:
  - http://edamontology.org/topic_2269
  - http://edamontology.org/topic_3270
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# cox-proportional-hazards-survival-analysis

## Summary

Per-cohort Cox proportional hazards modelling with age at blood sampling as the time scale, followed by fixed-effect meta-analysis and a forward-backward stepwise selection on Bonferroni-corrected P values, to identify circulating metabolic biomarkers independently associated with all-cause mortality.

## When to use

Use when cohort-level survival summary statistics (e.g. 226 Nightingale NMR metabolic biomarkers vs. all-cause mortality) must be combined across cohorts via fixed-effect meta-analysis to select an independent, significant biomarker set.

## When NOT to use

- The goal is clinical patient classification: the article notes the score built from per-cohort scaled biomarker values 'is not yet suitable for classification of patients in the clinic'; a concentration-unit score from individual-level data is needed instead.
- Biomarker values are already transformed/scaled — applying x+1, log-transform, and per-cohort scaling again would double-transform the data.
- The question is predictive performance benchmarking (C-statistic, IDI against conventional risk factors) rather than identification of independently associated biomarkers.

## Inputs

- Cohort-level Cox survival summary statistics for 226 metabolic biomarkers (e.g. from the BBMRI-omics atlas)
- Per-cohort biomarker measurements (EDTA plasma or serum, Nightingale NMR platform) with zeros requiring x+1 transformation
- Cohort metadata: age at blood sampling, sex, vital status/follow-up time, study-specific demographic and relatedness covariates
- Candidate biomarker subset list (63 biomarkers used for stepwise selection to prevent overfitting)

## Outputs

- Table of independently associated biomarkers (14 expected) with HR, 95% CI, P value, I2 heterogeneity statistic, and P(het) — e.g. table2_biomarker_meta_analysis.tsv
- Direction-of-effect labels per biomarker (lower risk: XXL-VLDL-L, S-HDL-L, VLDL-D, PUFA/FA, His, Leu, Val, Alb; higher risk: Glc, Lac, Ile, Phe, AcAce, GlycA)
- Per-cohort Cox regression estimates for all 226 biomarkers

## How to apply

Preprocess biomarkers as in the article: add 1 to biomarkers containing zeroes (x+1), log-transform, and scale to standard deviation units separately per cohort. Per cohort, fit a Cox proportional hazards model with age at blood sampling as the time scale, adjusted for sex and study-specific demographic/relatedness covariates. Combine cohort-level hazard ratios with a fixed-effect meta-analysis implemented in the R-package meta. Run the stepwise (forward-backward) selection on the 63-biomarker subset: in each round add the unselected biomarker with the lowest P value, remove any biomarker whose P rises above the Bonferroni threshold 2.21e-4 (0.05/226), and stop when all unselected biomarkers exceed the threshold. The rationale is to prevent overfitting while retaining only biomarkers whose association survives adjustment for all others; in the article this retains 14 biomarkers (citrate drops out due to limited additional contribution).

## Related tools

- **R** (Environment for per-cohort Cox proportional hazards fitting and the stepwise forward-backward selection procedure)
- **R-package meta** (Fixed-effect meta-analysis combining cohort-level biomarker-mortality hazard ratios in each selection round)

## Examples

```
Rscript -e 'library(meta); m <- metagen(TE=log(HR), seTE=se, studlab=cohort, sm="HR", method="fixed")'
```

## Evaluation signals

- Exactly 14 biomarkers are retained after the stepwise procedure, with citrate dropping out.
- Tabulated HRs and 95% CIs match published Table 2 values, e.g. GlycA HR 1.32 (95% CI 1.27-1.38), XXL-VLDL-L HR 0.80 (95% CI 0.75-0.85), Glc HR 1.16 (95% CI 1.13-1.19).
- All retained biomarkers have meta-analysis P below the Bonferroni threshold 2.21e-4 (0.05/226) at termination of the procedure.
- Each biomarker's direction of effect matches expectations: protective for XXL-VLDL lipids, small-HDL lipids, VLDL diameter, PUFA/FA, His, Leu, Val, Alb; harmful for Glc, Lac, Ile, Phe, AcAce, GlycA.
- Heterogeneity statistics (I2, P(het)) are reported for every retained biomarker.

## Limitations

- Selection is restricted to a 63-biomarker subset of the 226 measured to prevent overfitting; biomarkers outside this subset cannot enter the model.
- The targeted NMR platform captures 'only a fraction of the metabolites in the human serum'; more complete platforms were not compared.
- The biomarker score constructed from per-cohort scaled values is not directly usable in the clinic; concentration-unit weights from individual-level data are still needed.
- Risk-score weights were trained only on the Estonian Biobank cohort and tested only in FINRISK 1997, so generalization beyond these is untested.

## Evidence

- [methods] Preprocessing and model specification: [evidence span withheld — anchor status: not_found]
- [results] Stepwise selection: "we performed a stepwise (forward-backward) procedure based on successive rounds of meta-analyses"
- [methods] Significance threshold: "Biomarkers were considered significant when the P value was below the Bonferroni-adjusted threshold of 2.21 × 10−4 (0.05/226)"
- [discussion] Candidate subset rationale: "the selection of independently associated biomarkers was based on a subset of 63 biomarkers to prevent overfitting"
- [methods] Meta-analysis tooling: "the meta-analyses were performed using a fixed-effect model implemented in the R-package meta"
