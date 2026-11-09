---
name: benchmark-dose-calculation
description: Use when a metabolomics feature table contains samples measured across
  multiple dose levels of an exposure (e.g. in vitro assays or animal models) and
  you need fitted dose-response curves and per-feature benchmark dose (BMD) values
  for risk assessment.
when_to_use_negative:
- The dataset compares only two conditions or a single dose versus control — dose-response
  curve fitting requires multiple dose levels (the article notes experiments 'are
  often conducted at multiple dose levels')
- Input is raw MS spectra rather than a processed feature table — use the Spectra
  Processing module first to generate quantified features before dose-response modeling
edam_topics: []
tools:
- name: MetaboAnalyst 6.0
  role: Web platform whose Dose-Response Analysis module performs normalization, differential
    analysis to select dose-dependent features, linear/non-linear curve fitting, and
    benchmark dose calculation
- name: MetaboAnalystR 4.0
  role: R package containing the underlying functions of the MetaboAnalyst web server,
    enabling local reproduction of the dose-response/BMD workflow and replay of downloaded
    web command histories
  repo: https://github.com/xia-lab/MetaboAnalystR
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/nar/gkae253
    title: 'MetaboAnalyst 6.0: towards a unified platform for metabolomics data processing,
      analysis and interpretation'
  source_content_hash: sha256:d5942e2691950df41d38ea2d87d1209adce73cb2719fcd395f68c77be6030f25
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/xia-lab/MetaboAnalystR
  tools:
  - MetaboAnalyst 6.0
  - MetaboAnalystR 4.0
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# benchmark-dose-calculation

## Summary

Fits dose-response curves to metabolomics features measured across multiple exposure levels and derives per-feature benchmark dose (BMD) values, using the Dose-Response Analysis module introduced in MetaboAnalyst 6.0 for chemical risk assessment.

## When to use

Use when a metabolomics feature table contains samples measured across multiple dose levels of an exposure (e.g. in vitro assays or animal models) and you need fitted dose-response curves and per-feature benchmark dose (BMD) values for risk assessment.

## When NOT to use

- The dataset compares only two conditions or a single dose versus control — dose-response curve fitting requires multiple dose levels (the article notes experiments 'are often conducted at multiple dose levels')
- Input is raw MS spectra rather than a processed feature table — use the Spectra Processing module first to generate quantified features before dose-response modeling

## Inputs

- Feature intensity table (peak table) from a metabolomics dose-response experiment with samples spanning multiple dose levels (case study: four etomoxir doses on BT549 breast cancer cells)
- Sample metadata annotating each sample with its administered dose
- Optional: raw LC-MS data if spectra processing is needed upstream to produce the feature table

## Outputs

- Fitted dose-response curves per significant feature (linear and non-linear model fits)
- Per-feature benchmark dose (BMD) values table
- Dose-response curve plots for each modeled feature

## How to apply

Load the dose-response dataset (e.g. the published BT549 breast cancer cell dataset treated with four etomoxir doses) into MetaboAnalyst 6.0's Dose-Response Analysis module. First apply the module's built-in normalization and differential analysis to identify features with significant dose-dependent response. For each significant feature, fit dose-response curves against both linear and non-linear models, then compute the benchmark dose (BMD) from each fitted curve. Generate the fitted dose-response curve plots and the per-feature BMD table; validity is judged by agreement with published case study figures. The identical analysis can be reproduced locally in R, since MetaboAnalystR 4.0 exposes the R functions underlying the MetaboAnalyst web server, and users can replay the downloaded web command history for maximum reproducibility.

## Related tools

- **MetaboAnalyst 6.0** (Web platform whose Dose-Response Analysis module performs normalization, differential analysis to select dose-dependent features, linear/non-linear curve fitting, and benchmark dose calculation)
- **MetaboAnalystR 4.0** (R package containing the underlying functions of the MetaboAnalyst web server, enabling local reproduction of the dose-response/BMD workflow and replay of downloaded web command histories) — https://github.com/xia-lab/MetaboAnalystR

## Examples

```
devtools::install_github("xia-lab/MetaboAnalystR", build = TRUE, build_vignettes = FALSE)
```

## Evaluation signals

- The fitted dose-response curves and per-feature BMD values should reproduce the results shown in the published MetaboAnalyst 6.0 case study figures for the etomoxir BT549 dataset; agreement with those figures is the acceptance criterion
- Differential analysis after normalization should identify the features with significant dose-dependent response before any curve fitting is attempted
- Each significant feature yields both a fitted curve (from linear and non-linear candidate models) and a corresponding BMD derived from that fitted curve, not from raw intensities
- The per-feature BMD table contains an entry for every feature that passed the dose-dependency filter, and example feature-level BMD values match those reported in the case study
- When run via MetaboAnalystR, results should match the web server output for the same input, since the R package is synchronized with the web server

## Limitations

- The demonstrated case study uses only four etomoxir doses on BT549 cells, so performance with fewer or more dose levels is not benchmarked in this paper
- BMD estimation depends on prior identification of significant dose-dependent features; features failing the module's normalization/differential filtering receive no fitted curve or BMD
- Dose-response analysis supports risk assessment of chemical exposures, but the module is new in version 6.0 and broader validation beyond the showcased case study is not documented

## Evidence

- [abstract] a Dose-Response Analysis module for benchmark dose calculations: "a Dose-Response Analysis module for benchmark dose calculations"
- [intro] Dose-response experiments are often conducted at multiple dose levels using in vitro assays or animal models to calculate dose-response curves for risk assessment of chemical exposures: "The experiments are often conducted at multiple dose levels using in vitro assays or animal models to calculate dose-response curves for risk assessment of chemical exposures"
- [methods] MetaboAnalystR 4.0 contains the R functions and libraries underlying the popular MetaboAnalyst web server: "MetaboAnalystR 4.0 contains the R functions and libraries underlying the popular MetaboAnalyst web server, including metabolomic data analysis, visualization, and functional interpretation."
- [readme] users will be able to reproduce the same results from their local computers using the corresponding R command history downloaded from MetaboAnalyst web site: "After installing and loading the package, users will be able to reproduce the same results from their local computers using the corresponding R command history downloaded from MetaboAnalyst web site"
