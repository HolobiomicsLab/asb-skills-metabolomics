---
name: log-hazard-ratio-weight-computation
description: Use when building a weighted additive biomarker risk score from published
  Cox meta-analysis HRs (e.g., 14 Nightingale NMR biomarkers for all-cause mortality)
  and you need score weights plus a per-unit score HR.
when_to_use_negative:
- You need a clinically deployable score in concentration units — the paper explicitly
  states the score 'is not yet suitable for classification of patients in the clinic,
  since it is based on scaled biomarker values created separately for each cohort'
- Your goal is predictive performance comparison (C-statistic/IDI) of the score versus
  conventional risk factors in a held-out cohort such as FINRISK 1997, rather than
  constructing the weighted score itself
- You are fitting single-biomarker Cox associations rather than combining multiple
  biomarkers into an additive weighted score
tools:
- name: R
  role: Compute ln(HR) weights, apply the x+1 / log-transform / per-cohort scaling
    preprocessing, and build the weighted-sum score
- name: R-package meta
  role: Fixed-effect meta-analysis model from which the ln(hazard ratio) weights are
    derived
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-019-11311-9
    title: A metabolic profile of all-cause mortality risk identified in an observational
      study of 44,168 individuals
  source_content_hash: sha256:e22a9bcecc65c1aa745bdf85cf5a2f5a6d23330dd3c1ee39489e55b9946c10ae
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - R-package meta
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# log-hazard-ratio-weight-computation

## Summary

Constructs a metabolic biomarker mortality score by weighting log-transformed, per-cohort-scaled biomarker values with ln(hazard ratio) weights taken from fixed-effect survival meta-analysis estimates (Table 2 / BBMRI-omics atlas).

## When to use

Use when building a weighted additive biomarker risk score from published Cox meta-analysis HRs (e.g., 14 Nightingale NMR biomarkers for all-cause mortality) and you need score weights plus a per-unit score HR.

## When NOT to use

- You need a clinically deployable score in concentration units — the paper explicitly states the score 'is not yet suitable for classification of patients in the clinic, since it is based on scaled biomarker values created separately for each cohort'
- Your goal is predictive performance comparison (C-statistic/IDI) of the score versus conventional risk factors in a held-out cohort such as FINRISK 1997, rather than constructing the weighted score itself
- You are fitting single-biomarker Cox associations rather than combining multiple biomarkers into an additive weighted score

## Inputs

- Hazard ratios and 95% CIs for the 14 independently associated biomarkers from Table 2 of Deelen et al. 2019 (Nat Commun, DOI 10.1038/s41467-019-11311-9)
- Primary survival meta-analysis summary statistics from the BBMRI-omics atlas (http://bbmri.researchlumc.nl/atlas)
- Cohort-level biomarker matrices from the 12 cohorts (e.g., FINRISK 1997, Estonian Biobank, PROSPER, Rotterdam Study) with below-detection-limit zeroes flagged

## Outputs

- metabolic_biomarker_score_results.csv with 14 biomarker rows of ln(HR) weights matching Table 2
- Weighted-sum metabolic biomarker score per individual and its per-unit association (HR = 2.73, 95% CI: 2.60–2.86)
- Age-stratified (<60, 60–70, 70–80, 80–90, >90 years) and sex-stratified direction-of-effect consistency flags for all 14 biomarkers

## How to apply

Load each biomarker's hazard ratio and 95% CI from the fixed-effect survival meta-analysis (Table 2, or the summary statistics deposited in the BBMRI-omics atlas at http://bbmri.researchlumc.nl/atlas) and compute the weight as ln(HR), e.g. using the R-package 'meta' which the authors used for fixed-effect pooling. Preprocess biomarker values exactly as the paper does: add 1 to all biomarkers containing zeroes (x+1), log-transform, then scale to standard deviation units separately per study — scaling must be per-cohort to reproduce the reported associations. Multiply each scaled biomarker by its ln(HR) weight and sum to obtain the metabolic biomarker score. Compute or record the per-unit score HR (HR = 2.73, 95% CI 2.60–2.86 for a one-unit increase) and cross-check weights against the deposited summary statistics. The Cox models use age at blood sampling as the time scale, so any stratified recomputation (age groups <60, 60–70, 70–80, 80–90, >90; both sexes) must preserve that framing.

## Related tools

- **R** (Compute ln(HR) weights, apply the x+1 / log-transform / per-cohort scaling preprocessing, and build the weighted-sum score)
- **R-package meta** (Fixed-effect meta-analysis model from which the ln(hazard ratio) weights are derived)

## Examples

```
weights <- log(HR_table2); score <- as.vector(scale(log(biomarkers + 1)) %*% weights)
```

## Evaluation signals

- Output table contains exactly 14 biomarker rows with ln(HR) weights matching Table 2
- Reproduced score HR equals 2.73 (95% CI: 2.60–2.86) for a one-unit increase
- Below-detection-limit zeroes were incremented by 1 (x+1) before log-transformation, and scaling to SD units was done separately per study — not pooled
- All 14 biomarkers show consistent association directions across the five age strata (<60, 60–70, 70–80, 80–90, >90) and both sexes, matching Supplementary Data 4
- ln(HR) weights cross-check against the deposited BBMRI-omics atlas summary statistics

## Limitations

- The score is not suitable for clinical patient classification because it is based on per-cohort scaled biomarker values; a concentration-unit score from individual-level data is still needed
- Score weights were trained only on the Estonian Biobank cohort and predictive ability tested only in FINRISK 1997, so generalization beyond these cohorts is unverified
- Only summary statistics are available (BBMRI-omics atlas); no code repository is mentioned, so the exact scaling implementation must be re-derived

## Evidence

- [discussion] the log-transformed and scaled biomarkers were multiplied by their weights, based on the meta-analyses results (i.e., ln(hazard ratio) from Table 2), and subsequently summed: "the log-transformed and scaled biomarkers were multiplied by their weights, based on the meta-analyses results (i.e., ln(hazard ratio) from Table 2), and subsequently summed"
- [discussion] a value of one was added to all biomarkers containing zeroes (i.e., x + 1) ... all biomarkers were log-transformed and scaled to standard deviation units, separately per study: [evidence span withheld — anchor status: not_found]
- [discussion] the meta-analyses were performed using a fixed-effect model implemented in the R-package meta: "the meta-analyses were performed using a fixed-effect model implemented in the R-package meta"
- [discussion] The summary statistics of our primary survival meta-analysis have been made publically available in the BBMRI -omics atlas: "The summary statistics of our primary survival meta-analysis have been made publically available in the BBMRI -omics atlas"
- [discussion] a Cox proportional hazards model with age at blood sampling as the time scale, was used to determine the associations of the biomarkers with all-cause mortality: "a Cox proportional hazards model with age at blood sampling as the time scale, was used to determine the associations of the biomarkers with all-cause mortality"
