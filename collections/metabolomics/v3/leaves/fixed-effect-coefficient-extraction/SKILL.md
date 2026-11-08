---
name: fixed-effect-coefficient-extraction
description: Use when per-metabolite LMMs have been fitted with patient metadata fixed
  effects (e.g. 'Sex','Age','BMI','Stage','Location','Tissue') and you need the coefficient
  and significance of a specific fixed effect for a metabolite.
when_to_use_negative:
- Data have not been log2-transformed and LMM-fitted — coefficient extraction presupposes
  converged per-metabolite lmm2met fits
- The question concerns the readjusted metabolite abundance matrix itself (fitMet$fittedDat)
  rather than the fixed-effect coefficients
- You need random-effect (subject intercept) estimates rather than patient-metadata
  fixed-effect coefficients
edam_operation: http://edamontology.org/operation_3657
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_2269
tools:
- name: lmm2met
  role: fitLmm fits per-metabolite LMMs and getFixCoeff extracts the fixed-effect
    coefficient and chi-square significance table
  repo: https://github.com/kwanjeeraw/lmm2met
- name: lme4
  role: underlying linear model-fitting procedure on which fitLmm's per-metabolite
    fits are implemented
- name: devtools
  role: used to install lmm2met from GitHub via install_github
provenance:
  source_task_ids:
  - task_004
  - task_006
  source_papers:
  - doi: 10.1016/j.csbj.2019.04.009
    title: Accounting for biological variation with linear mixed-effects modelling
      improves the quality of clinical metabolomics data
  source_content_hash: sha256:15665e62f95db1116088a84cb6638e2ef5b6f82dfb8bffad2d581344b696189a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/kwanjeeraw/lmm2met
  tools:
  - R
  - devtools
  - lme4
  - lmm2met
  edam_operation: http://edamontology.org/operation_3657
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# fixed-effect-coefficient-extraction

## Summary

Extracts the estimated fixed-effect coefficients and chi-square-test significances for patient metadata factors (e.g. Tissue, Sex, Age, BMI, Stage, Location) from lmm2met LMM fits of log2-transformed clinical metabolomics data, as with the cholesterol VAT-vs-SAT coefficient (0.05, p = .05) reported in Table S-2.

## When to use

Use when per-metabolite LMMs have been fitted with patient metadata fixed effects (e.g. 'Sex','Age','BMI','Stage','Location','Tissue') and you need the coefficient and significance of a specific fixed effect for a metabolite.

## When NOT to use

- Data have not been log2-transformed and LMM-fitted — coefficient extraction presupposes converged per-metabolite lmm2met fits
- The question concerns the readjusted metabolite abundance matrix itself (fitMet$fittedDat) rather than the fixed-effect coefficients
- You need random-effect (subject intercept) estimates rather than patient-metadata fixed-effect coefficients

## Inputs

- fitLmm output object fitted on a log2-transformed metabolomics dataframe with metadata columns plus metabolite abundance columns (e.g. 158 lipids, 59 CRC patients, paired VAT/SAT)
- fixed-effect specification fix=c('Sex','Age','BMI','Stage','Location','Tissue') and random='(1|Id)'
- target metabolite name (e.g. cholesterol) and fixed-effect term (e.g. TissueVAT)

## Outputs

- fixed-effect coefficient table per metabolite with chi-square-test significances (from getFixCoeff)
- coefficient/significance result file, e.g. cholesterol_vat_lmm_result.csv
- fixed-effect significance heatmap via plot(fitMet, type='coeff') and fixed_effect_coefficients.csv

## How to apply

After loading the log2-transformed metabolite matrix (e.g. 158 lipids from 59 paired CRC VAT/SAT samples from Metabolomics Workbench ST000081) and fitting per-metabolite LMMs with fitLmm (fixed effects c('Sex','Age','BMI','Stage','Location','Tissue'), random intercept '(1|Id)' on lme4), call getFixCoeff on the fitted object to retrieve coefficients and chi-square-test significances for each metabolite. Identify the row of the metabolite of interest (e.g. cholesterol) and read the coefficient of the factor level of interest (e.g. the Tissue VAT term) and its p-value. Decision point: validate against published values — the cholesterol VAT coefficient should equal 0.05 with p-value 0.05, matching Table S-2. The same coefficient tables can be visualized with plot(fitMet, type='coeff') to inspect fixed-effect significance heatmaps before writing results (e.g. cholesterol_vat_lmm_result.csv).

## Related tools

- **lmm2met** (fitLmm fits per-metabolite LMMs and getFixCoeff extracts the fixed-effect coefficient and chi-square significance table) — https://github.com/kwanjeeraw/lmm2met
- **lme4** (underlying linear model-fitting procedure on which fitLmm's per-metabolite fits are implemented)
- **devtools** (used to install lmm2met from GitHub via install_github)

## Examples

```
fitMet <- lmm2met::fitLmm(data, fix=c('Sex','Age','BMI','Stage','Location','Tissue'), random='(1|Id)'); coeff <- lmm2met::getFixCoeff(fitMet)
```

## Evaluation signals

- Cholesterol VAT coefficient equals 0.05 and chi-square p-value equals 0.05, matching Table S-2 of the paper
- getFixCoeff returns coefficients and chi-square-test significances for every metabolite in the input matrix (e.g. all 158 lipids)
- Inspecting fitMet$completeMod[[1]] confirms the first per-metabolite LMM converged
- Coefficient heatmap via plot(fitMet, type='coeff') renders without error and matches exported fixed_effect_coefficients.csv
- Output CSV (e.g. cholesterol_vat_lmm_result.csv) contains exactly the target metabolite's coefficient and p-value with consistent formatting

## Limitations

- Coefficient estimates depend on convergence of each per-metabolite LMM; non-convergent fits yield unreliable fixed-effect terms
- Increased sensitivity of the LMM processing step carries the potential introduction of false positives
- Reported performance claims are at the current (preliminary) stage and further experiments are recommended to validate the results

## Evidence

- [methods] Fixed effects 'Sex','Age','BMI','Stage','Location','Tissue' and random intercept '(1|Id)': [evidence span withheld — anchor status: not_found]
- [methods] getFixCoeff returns per-metabolite coefficients and significances: [evidence span withheld — anchor status: not_found]
- [results] Cholesterol VAT coefficient and p-value validation target: [evidence span withheld — anchor status: not_found]
- [abstract] LMM formulation with metadata as fixed effects: [evidence span withheld — anchor status: not_found]
- [readme] Install lmm2met from GitHub: "Install lmm2met using the following commands"
