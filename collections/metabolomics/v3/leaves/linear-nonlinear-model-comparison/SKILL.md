---
name: linear-nonlinear-model-comparison
description: Use when a metabolomics feature table spans several dose levels of a
  compound (e.g. four etomoxir doses on BT549 cells) and the goal is fitted dose-response
  curves plus per-feature benchmark dose (BMD) values for risk assessment.
when_to_use_negative:
- The study has only two conditions or a single dose with control — use standard differential/uni-variate
  statistical analysis instead of dose-response fitting
- Input is still raw LC-MS spectra — run the MetaboAnalyst 6.0 Spectra Processing
  module (MS1/MS2 peak detection, asari algorithm) first to obtain a feature table
- The aim is risk assessment of chemical exposures from animal-model data at dose
  levels incompatible with the module's in vitro-cell-line-style design (the case
  study is demonstrated on four etomoxir doses in BT549 cells)
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0203
tools:
- name: MetaboAnalyst 6.0
  role: Web platform hosting the Dose-Response Analysis module that performs normalization,
    differential analysis, linear/non-linear curve fitting, and benchmark dose (BMD)
    calculation
  repo: https://github.com/xia-lab/MetaboAnalystR
- name: MetaboAnalystR 4.0
  role: R package with the underlying functions of the MetaboAnalyst web server, enabling
    scripted reproduction of the same dose-response/BMD analysis locally
  repo: https://github.com/xia-lab/MetaboAnalystR
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/nar/gkae253
    title: 'MetaboAnalyst 6.0: towards a unified platform for metabolomics data processing,
      analysis and interpretation'
  source_content_hash: sha256:4b4403ef850e20f511a385c38d1a66ae628884dd84f90081346d53a7c7af34a1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/xia-lab/MetaboAnalystR
  tools:
  - MetaboAnalyst 6.0
  - MetaboAnalystR 4.0
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# Linear-nonlinear model comparison

## Summary

Fits metabolomics features measured across multiple dose levels to both linear and non-linear dose-response models and derives per-feature benchmark dose (BMD) values, as implemented in the MetaboAnalyst 6.0 Dose-Response Analysis module and demonstrated on the published etomoxir BT549 breast cancer cell dataset.

## When to use

Use when a metabolomics feature table spans several dose levels of a compound (e.g. four etomoxir doses on BT549 cells) and the goal is fitted dose-response curves plus per-feature benchmark dose (BMD) values for risk assessment.

## When NOT to use

- The study has only two conditions or a single dose with control — use standard differential/uni-variate statistical analysis instead of dose-response fitting
- Input is still raw LC-MS spectra — run the MetaboAnalyst 6.0 Spectra Processing module (MS1/MS2 peak detection, asari algorithm) first to obtain a feature table
- The aim is risk assessment of chemical exposures from animal-model data at dose levels incompatible with the module's in vitro-cell-line-style design (the case study is demonstrated on four etomoxir doses in BT549 cells)

## Inputs

- Metabolomics feature concentration/intensity table measured across multiple (e.g. four) dose levels of an exposure compound, with sample metadata indicating dose group
- Published etomoxir dose-response dataset from BT549 breast cancer cells (case study dataset used by the MetaboAnalyst 6.0 authors)

## Outputs

- Fitted dose-response curve plots per significant feature (linear and non-linear model fits)
- Per-feature benchmark dose (BMD) table derived from the fitted curves
- List of features passing the module's normalization and differential analysis as significantly dose-responsive

## How to apply

Load the multi-dose metabolomics dataset (here, the published BT549 breast cancer cell dataset with four etomoxir doses) into the MetaboAnalyst 6.0 Dose-Response Analysis module, or drive the same analysis locally with MetaboAnalystR 4.0, which contains the R functions underlying the web server. First apply the module's built-in normalization and differential analysis to identify features with a significant dose-dependent response. For each significant feature, fit dose-response curves against both linear and non-linear models rather than assuming a single functional form, since in vitro dose-response experiments at multiple dose levels can exhibit non-linear cause-and-effect relationships. Compute a per-feature benchmark dose (BMD) from the fitted curve for each feature. Generate the fitted dose-response curve plots and the per-feature BMD table. Validate by comparing your figures and BMD values against the published MetaboAnalyst 6.0 case study figures for the etomoxir BT549 dataset — agreement with those figures is the acceptance criterion.

## Related tools

- **MetaboAnalyst 6.0** (Web platform hosting the Dose-Response Analysis module that performs normalization, differential analysis, linear/non-linear curve fitting, and benchmark dose (BMD) calculation) — https://github.com/xia-lab/MetaboAnalystR
- **MetaboAnalystR 4.0** (R package with the underlying functions of the MetaboAnalyst web server, enabling scripted reproduction of the same dose-response/BMD analysis locally) — https://github.com/xia-lab/MetaboAnalystR

## Examples

```
devtools::install_github("xia-lab/MetaboAnalystR", build = TRUE, build_vignettes = FALSE)
```

## Evaluation signals

- Fitted dose-response curve plots for the etomoxir BT549 dataset match the published MetaboAnalyst 6.0 case study figures
- Example per-feature BMD value computed from the fitted curve agrees with the BMD reported in the case study
- Only features identified as significantly dose-responsive by the module's built-in normalization and differential analysis enter the curve-fitting and BMD stage
- Local MetaboAnalystR output reproduces the web server results, since the package is synchronized with the MetaboAnalyst web server and users can replay the downloaded R command history

## Limitations

- The published demonstration uses only the four-dose etomoxir BT549 breast cancer cell dataset, so behavior on other dose counts or exposure designs is not shown in this case study
- Fitting is restricted to the linear and non-linear models implemented in the module's dose-response modeling; alternative functional forms are not covered here
- Compatibility of the newest MetaboAnalystR version with latest R is still under evaluation (the README recommends R base with version > 3.6.1)
- The source repository has no changelog, so version-to-version changes in the dose-response implementation are not traceable from the repo alone

## Evidence

- [abstract] Dose-Response Analysis module showcased on BT549 etomoxir case study: "a Dose-Response Analysis module for benchmark dose calculations"
- [intro] Dose-response experiments rationale: "Dose-response experiments are often performed to further quantify cause-and-effect relationships. The experiments are often conducted at multiple dose levels using in vitro assays or animal models to"
- [methods] MetaboAnalystR underlying functions: "MetaboAnalystR 4.0 contains the R functions and libraries underlying the popular MetaboAnalyst web server, including metabolomic data analysis, visualization, and functional interpretation."
- [readme] Reproducibility via synchronized package: "After installing and loading the package, users will be able to reproduce the same results from their local computers using the corresponding R command history downloaded from MetaboAnalyst web site"
