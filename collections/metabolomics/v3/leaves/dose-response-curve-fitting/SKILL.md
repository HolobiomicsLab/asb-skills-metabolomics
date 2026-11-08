---
name: dose-response-curve-fitting
description: Use when metabolomics feature abundances (e.g., LC-MS peak tables) are
  measured across multiple dose levels of an exposure and you need fitted dose-response
  curves and per-feature benchmark dose (BMD) values for chemical risk assessment.
when_to_use_negative:
- The data contain only a single dose vs. control comparison (no multi-level dose
  gradient), so no dose-response curve can be fitted and plain differential analysis
  should be used instead
- The input is raw MS spectra rather than a processed feature table — use the Spectra
  Processing module (asari/OptiLCMS) first; this skill assumes normalized, dose-annotated
  features
- The goal is causal metabolite-phenotype inference (Mendelian randomization) or multi-factor
  statistical analysis rather than exposure dose-response modeling
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: MetaboAnalyst 6.0
  role: Web platform whose Dose-Response Analysis module performs normalization, differential
    analysis, dose-response curve fitting against linear and non-linear models, and
    benchmark dose calculation (demonstrated on the etomoxir BT549 dataset)
- name: MetaboAnalystR 4.0
  role: R package containing the functions underlying the MetaboAnalyst web server,
    enabling local, reproducible execution of the dose-response workflow
  repo: https://github.com/xia-lab/MetaboAnalystR
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/nar/gkae253
    title: 'MetaboAnalyst 6.0: towards a unified platform for metabolomics data processing,
      analysis and interpretation'
  source_content_hash: sha256:08a345f9456afd2f3c64ba5f46b00054837fab7cd30639acc0c57c6f4f8b91e1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/xia-lab/MetaboAnalystR
  tools:
  - MetaboAnalyst 6.0
  - MetaboAnalystR 4.0
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# dose-response-curve-fitting

## Summary

Fit dose-response curves to metabolomics features measured across multiple exposure levels and compute per-feature benchmark dose (BMD) values using the MetaboAnalyst 6.0 Dose-Response Analysis module, exemplified by the etomoxir BT549 breast cancer cell case study.

## When to use

Use when metabolomics feature abundances (e.g., LC-MS peak tables) are measured across multiple dose levels of an exposure and you need fitted dose-response curves and per-feature benchmark dose (BMD) values for chemical risk assessment.

## When NOT to use

- The data contain only a single dose vs. control comparison (no multi-level dose gradient), so no dose-response curve can be fitted and plain differential analysis should be used instead
- The input is raw MS spectra rather than a processed feature table — use the Spectra Processing module (asari/OptiLCMS) first; this skill assumes normalized, dose-annotated features
- The goal is causal metabolite-phenotype inference (Mendelian randomization) or multi-factor statistical analysis rather than exposure dose-response modeling

## Inputs

- Metabolomics feature abundance table measured across multiple dose levels (e.g., the published etomoxir BT549 breast cancer cell dataset with four doses)
- Sample metadata specifying dose/exposure level per sample
- Normalization parameters accepted by the MetaboAnalyst 6.0 Dose-Response Analysis module

## Outputs

- Fitted dose-response curves (linear and non-linear models) per significant feature
- Per-feature benchmark dose (BMD) values derived from the fitted curves
- Fitted dose-response curve plots and a per-feature BMD table

## How to apply

Load a dose-response dataset (e.g., the published BT549 breast cancer cell dataset with four etomoxir doses) into MetaboAnalyst 6.0's Dose-Response Analysis module. First apply the module's built-in normalization and differential analysis to identify features showing significant dose-dependent response. For each significant feature, fit a dose-response curve against linear and non-linear models, selecting the fitted curve that models the feature's dose-dependent behavior. Compute the benchmark dose (BMD) for each feature directly from its fitted curve. Finally, generate the fitted dose-response curve plots and the per-feature BMD table, and compare both against the published case study figures as the acceptance criterion.

## Related tools

- **MetaboAnalyst 6.0** (Web platform whose Dose-Response Analysis module performs normalization, differential analysis, dose-response curve fitting against linear and non-linear models, and benchmark dose calculation (demonstrated on the etomoxir BT549 dataset))
- **MetaboAnalystR 4.0** (R package containing the functions underlying the MetaboAnalyst web server, enabling local, reproducible execution of the dose-response workflow) — https://github.com/xia-lab/MetaboAnalystR

## Evaluation signals

- The fitted dose-response curves and per-feature BMD values reproduce the results shown in the published MetaboAnalyst 6.0 case study figures for the etomoxir BT549 dataset — agreement with those figures is the acceptance criterion
- Only features identified as having significant dose-dependent response (via the module's built-in normalization and differential analysis) receive fitted curves and BMD values
- Each reported BMD is derived from the corresponding feature's fitted curve, not from raw abundance values
- The per-feature BMD table and curve plots are generated as paired outputs for the same feature set

## Limitations

- The module is designed for dose-response experiments conducted at multiple dose levels (in vitro assays or animal models); a two-group design without a dose gradient is out of scope
- The acceptance benchmark is visual/agreement comparison against the published case study figures rather than an automated numerical tolerance
- Results depend on the module's built-in normalization choices; features not passing the differential-analysis significance screen get no curve or BMD

## Evidence

- [abstract] Dose-response analysis module exists in v6.0: "a Dose-Response Analysis module for benchmark dose calculations"
- [intro] Dose-response experiments conducted at multiple dose levels for risk assessment: "The experiments are often conducted at multiple dose levels using in vitro assays or animal models to calculate dose-response curves for risk assessment of chemical exposures"
- [methods] MetaboAnalystR contains the R functions underlying the web server: "MetaboAnalystR 4.0 contains the R functions and libraries underlying the popular MetaboAnalyst web server, including metabolomic data analysis, visualization, and functional interpretation."
- [readme] Reproducibility via synchronized package and R command history: "users will be able to reproduce the same results from their local computers using the corresponding R command history downloaded from MetaboAnalyst web site"
