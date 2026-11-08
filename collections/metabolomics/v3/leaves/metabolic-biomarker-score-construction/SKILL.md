---
name: metabolic-biomarker-score-construction
description: Use when mortality-associated metabolic biomarkers and their meta-analysis
  hazard ratios are available and you need to reproduce a weighted, per-unit risk
  score plus age/sex stratified consistency checks.
when_to_use_negative:
- If the goal is clinical patient classification — the paper explicitly states the
  score 'is not yet suitable for classification of patients in the clinic, since it
  is based on scaled biomarker values created separately for each cohort'.
- If the analysis is prediction (C-statistic/IDI evaluation in FINRISK 1997) rather
  than score construction — use the prediction-skill workflow instead.
- If biomarker values are already on concentration units — the score here is defined
  on per-cohort SD-scaled values, so applying it to raw concentrations changes its
  meaning.
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_2269
- http://edamontology.org/topic_3053
tools:
- name: R
  role: Computes ln(HR) weights, applies the x+1/log/SD-scaling preprocessing, and
    evaluates the weighted-sum metabolic biomarker score
- name: R-package meta
  role: Fixed-effect meta-analysis pooling that produces the hazard ratios whose logarithms
    serve as score weights
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-019-11311-9
    title: A metabolic profile of all-cause mortality risk identified in an observational
      study of 44,168 individuals
  source_content_hash: sha256:16e7cc0712848e1ac491d7de7cfacd403e0b9eebf8dba0bd927eaa4aa91163c1
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - R-package meta
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_2269
  - http://edamontology.org/topic_3053
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# metabolic-biomarker-score-construction

## Summary

Constructs a weighted metabolic biomarker score for all-cause mortality by multiplying per-cohort log-transformed, standard-deviation-scaled NMR biomarker values by ln(hazard ratio) weights (from Table 2 fixed-effect meta-analysis) and summing them, yielding the reported per-unit score HR = 2.73 (95% CI: 2.60–2.86).

## When to use

Use when mortality-associated metabolic biomarkers and their meta-analysis hazard ratios are available and you need to reproduce a weighted, per-unit risk score plus age/sex stratified consistency checks.

## When NOT to use

- If the goal is clinical patient classification — the paper explicitly states the score 'is not yet suitable for classification of patients in the clinic, since it is based on scaled biomarker values created separately for each cohort'.
- If the analysis is prediction (C-statistic/IDI evaluation in FINRISK 1997) rather than score construction — use the prediction-skill workflow instead.
- If biomarker values are already on concentration units — the score here is defined on per-cohort SD-scaled values, so applying it to raw concentrations changes its meaning.

## Inputs

- Table 2 hazard ratios and 95% CIs for the 14 independently associated biomarkers (fixed-effect meta-analysis)
- BBMRI-omics atlas primary survival meta-analysis summary statistics (http://bbmri.researchlumc.nl/atlas)
- EDTA plasma or serum NMR metabolomics measurements (Nightingale Health platform) preprocessed as x+1, log-transformed, and scaled to SD units per cohort
- Supplementary Data 4 age- and sex-stratified association results
- Cohort metadata: age at blood sampling, sex, and vital status for Cox models

## Outputs

- metabolic_biomarker_score_results.csv with 14 biomarker rows, ln(HR) weights, score HR = 2.73 (CI 2.60–2.86)
- Per-cohort scaled biomarker matrix ready for weighted-sum scoring
- Age-stratified (<60, 60–70, 70–80, 80–90, >90 years) and sex-stratified direction-of-effect consistency flags

## How to apply

Start from the 14 biomarker hazard ratios and 95% CIs in Table 2 of Deelen et al. (Nature Communications, DOI 10.1038/s41467-019-11311-9), or the summary statistics in the BBMRI-omics atlas (http://bbmri.researchlumc.nl/atlas). In R, compute each biomarker's score weight as ln(HR) from the fixed-effect meta-analysis estimates (pooling done with the R-package meta). Apply the paper's preprocessing convention exactly: add 1 to biomarkers containing zeroes (x + 1), log-transform, and scale to standard deviation units separately per cohort, so the 'weighted sum of scaled biomarkers' formula is reproduced. Multiply each scaled biomarker by its ln(HR) weight and sum to obtain the metabolic biomarker score; record the per-unit mortality association (HR = 2.73, 95% CI: 2.60–2.86, P < 1.00 × 10−132). The underlying Cox proportional hazards models use age at blood sampling as the time scale, and consistency is verified across age strata <60, 60–70, 70–80, 80–90, >90 years and both sexes (Supplementary Data 4), where all 14 biomarkers show consistent association directions.

## Related tools

- **R** (Computes ln(HR) weights, applies the x+1/log/SD-scaling preprocessing, and evaluates the weighted-sum metabolic biomarker score)
- **R-package meta** (Fixed-effect meta-analysis pooling that produces the hazard ratios whose logarithms serve as score weights)

## Examples

```
Rscript -e 'weights <- log(tab2$HR); score <- as.numeric(scaled14 %*% weights); write.csv(data.frame(biomarker=tab2$name, lnHR=weights), "metabolic_biomarker_score_results.csv", row.names=FALSE)'
```

## Evaluation signals

- metabolic_biomarker_score_results.csv contains exactly 14 biomarker rows
- Each ln(HR) weight matches Table 2 of the article and cross-checks against the BBMRI-omics atlas summary statistics
- Score-level association reproduces HR = 2.73 with 95% CI 2.60–2.86
- All 14 biomarkers show consistent association directions across all five age strata (<60, 60–70, 70–80, 80–90, >90 years) and both sexes, as in Supplementary Data 4
- Preprocessing check: biomarkers containing zeroes received x+1 before log-transformation and were scaled to SD units separately per study

## Limitations

- The score is based on per-cohort scaled biomarker values and is not suitable for classification of patients in the clinic
- A score based on concentration units from individual-level data is still needed for clinical research
- NMR platform captures only a fraction of the metabolites in human serum; more complete platforms were not compared
- Risk score weights were trained on the Estonian Biobank cohort (Supplementary Table 4) and tested only in the FINRISK 1997 cohort

## Evidence

- [discussion] Score construction: "the log-transformed and scaled biomarkers were multiplied by their weights, based on the meta-analyses results (i.e., ln(hazard ratio) from Table 2), and subsequently summed"
- [discussion] Preprocessing convention: [evidence span withheld — anchor status: not_found]
- [discussion] Cox model framing: "a Cox proportional hazards model with age at blood sampling as the time scale, was used to determine the associations of the biomarkers with all-cause mortality"
- [discussion] Age stratification groups: "samples were divided into age groups of <60, 60–70, 70–80, 80–90, and >90 years"
- [abstract] Overall consistency claim: "Overall, these associations are similar in men and women and across different age strata."
