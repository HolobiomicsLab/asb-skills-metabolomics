---
name: chi-square-significance-testing
description: Use when a linear mixed-effects model has been fitted per metabolite
  (e.g. with fixed effects 'Sex','Age','BMI','Stage','Location','Tissue' and random
  intercept '(1|Id)') and you need chi-square p-values for fixed-effect terms such
  as a VAT-vs-SAT tissue effect.
when_to_use_negative:
- Raw, non-LMM-fitted data — before LMM fitting, cholesterol signals were randomly
  mixed and indistinguishable in the M0 dataset, so the tissue-effect test is not
  meaningful
- Inputs lacking paired/subject metadata needed for the random intercept '(1|Id)'
  and the fixed-effect design (Sex, Age, BMI, Stage, Location, Tissue)
edam_operation: http://edamontology.org/operation_3664
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3520
tools:
- name: lmm2met
  role: Provides fitLmm for per-metabolite LMM fitting and getFixCoeff for extracting
    fixed-effect coefficients with chi-square-test significances
  repo: https://github.com/kwanjeeraw/lmm2met
- name: lme4
  role: Underlying linear model-fitting procedure on which the lmm2met LMM fitting
    is implemented
- name: R
  role: Runtime environment for installing lmm2met via devtools and running the fitting/significance
    extraction
  repo: https://www.r-project.org/
- name: devtools
  role: Used to install the lmm2met package from GitHub via install_github
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.csbj.2019.04.009
    title: Accounting for biological variation with linear mixed-effects modelling
      improves the quality of clinical metabolomics data
  source_content_hash: sha256:03a7522af537f26b4257cf6932499db786c287d50d38059c95fa8c2afcc957b2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/kwanjeeraw/lmm2met
  tools:
  - R
  - devtools
  - lme4
  - lmm2met
  edam_operation: http://edamontology.org/operation_3664
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# chi-square-significance-testing

## Summary

Assess significance of LMM fixed-effect coefficients (e.g. the Tissue/VAT term in the CRC adipose lipid data) via chi-square tests returned by lmm2met's getFixCoeff, enabling metabolite-level hypothesis testing such as the cholesterol VAT-vs-SAT contrast reported in Table S-2.

## When to use

Use when a linear mixed-effects model has been fitted per metabolite (e.g. with fixed effects 'Sex','Age','BMI','Stage','Location','Tissue' and random intercept '(1|Id)') and you need chi-square p-values for fixed-effect terms such as a VAT-vs-SAT tissue effect.

## When NOT to use

- Raw, non-LMM-fitted data — before LMM fitting, cholesterol signals were randomly mixed and indistinguishable in the M0 dataset, so the tissue-effect test is not meaningful
- Inputs lacking paired/subject metadata needed for the random intercept '(1|Id)' and the fixed-effect design (Sex, Age, BMI, Stage, Location, Tissue)

## Inputs

- Log2-transformed metabolomics matrix (e.g. 158 lipids from 59 CRC patients, paired VAT/SAT adipose samples) from the lmm2met adipose dataset or Metabolomics Workbench ST000081/PR000058
- Patient metadata table with covariates used as fixed effects (Sex, Age, BMI, Stage, Location, Tissue) and patient Id
- LMM model specification: fixed effects c('Sex','Age','BMI','Stage','Location','Tissue') and random intercept '(1|Id)'

## Outputs

- Fixed-effect coefficient table per metabolite (coefficients plus chi-square-test significances) from getFixCoeff
- Target-metabolite result (e.g. cholesterol VAT coefficient = 0.05, chi-square p-value = 0.05) written to cholesterol_vat_lmm_result.csv

## How to apply

After fitting an LMM to each log2-transformed metabolite with fitLmm (built on lme4), call getFixCoeff to extract the fixed-effect coefficient table, which returns coefficients and chi-square-test significances for each metabolite. Locate the term of interest (e.g. the Tissue/VAT coefficient) for a target metabolite such as cholesterol and read off its coefficient estimate and chi-square p-value. In the paper's CRC adipose example (158 lipids, 59 colorectal cancer patients, paired VAT/SAT from study ST000081/PR000058), this recovered a cholesterol VAT coefficient of 0.05 with p-value 0.05, matching Table S-2. Write the extracted values to a results file (e.g. cholesterol_vat_lmm_result.csv) and compare against the reported paper values as validation. Note that before LMM fitting, cholesterol signals in the M0 dataset were randomly mixed and indistinguishable by PLS-DA, so the chi-square significance emerges only after the LMM processing step.

## Related tools

- **lmm2met** (Provides fitLmm for per-metabolite LMM fitting and getFixCoeff for extracting fixed-effect coefficients with chi-square-test significances) — https://github.com/kwanjeeraw/lmm2met
- **lme4** (Underlying linear model-fitting procedure on which the lmm2met LMM fitting is implemented)
- **R** (Runtime environment for installing lmm2met via devtools and running the fitting/significance extraction) — https://www.r-project.org/
- **devtools** (Used to install the lmm2met package from GitHub via install_github)

## Examples

```
install_github('kwanjeeraw/lmm2met'); fit <- fitLmm(data, metadata, fixedEffects=c('Sex','Age','BMI','Stage','Location','Tissue'), randomEffects='(1|Id)'); coefTable <- getFixCoeff(fit)
```

## Evaluation signals

- The extracted cholesterol VAT coefficient equals 0.05 (Table S-2)
- The extracted cholesterol chi-square p-value equals 0.05 (Table S-2)
- getFixCoeff returns a coefficient row for every metabolite in the 158-lipid matrix with both coefficient and significance columns populated
- Output file cholesterol_vat_lmm_result.csv contains the Tissue (VAT) coefficient and chi-square p-value for cholesterol

## Limitations

- Increased sensitivity after LMM processing carries the potential introduction of false positives
- Further experiments are recommended to validate the results (validation gap)
- Significances are chi-square tests on fixed effects; the paper qualifies its performance claims as being at a current (preliminary) stage

## Evidence

- [other] Extract the fixed-effect coefficient table with getFixCoeff, which returns coefficients and chi-square-test significances for each metabolite.: [evidence span withheld — anchor status: not_found]
- [methods] The package was implemented on a linear model-fitting procedure of the lme4 package: "The package was implemented on a linear model-fitting procedure of the lme4 package"
- [results] After LMM fitting, cholesterol in VAT was more abundant than SAT, with a VAT coefficient of 0.05 and a p-value of .05 (Table S-2).: [evidence span withheld — anchor status: not_found]
- [results] Before LMM fitting, cholesterol signals were randomly mixed and indistinguishable: "Before LMM fitting, cholesterol signals were randomly mixed and indistinguishable"
- [discussion] albeit with the potential introduction of false positives: "albeit with the potential introduction of false positives"
- [readme] Install lmm2met using the following commands ... install_github('kwanjeeraw/lmm2met'): "Install lmm2met using the following commands"
