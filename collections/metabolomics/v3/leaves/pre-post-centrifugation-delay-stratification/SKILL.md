---
name: pre-post-centrifugation-delay-stratification
description: Use when plasma/serum samples carry time-stamp-derived pre- and post-centrifugation
  delay metadata and you need to model or validate metabolite stability, or generate
  performance reports per delay type.
when_to_use_negative:
- Samples lack time-stamp-derived pre- or post-centrifugation delay information —
  the stratification and PRIMA performance report cannot be computed
- Analytes are known not to follow the linear mixed-model response (e.g. glucose in
  EDTA and LiHep plasma showed unexpectedly large MPE at later time points), so model-based
  stability claims would be misleading
- The goal is pass/fail sample QC only (Bruker BiobankQC provides that basic output);
  PRIMA stratification is for exploring delay effects and delay-specific modeling
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_3360
- http://edamontology.org/topic_3337
tools:
- name: PRIMA Panel
  role: Open-source R Shiny tool that stratifies stability time-points by pre-/post-centrifugation
    delay, computes delay times from date+time stamps, and generates performance reports
    from uploaded pre-analytical tables
  repo: github.com/funkam/PRIMA
- name: Shiny
  role: Framework hosting the PRIMA Panel app (also permanently hosted on Shinyapps.io)
    for interactive exploration and report download
  repo: github.com/funkam/PRIMA
- name: R
  role: Implementation language for the linear mixed models and PRIMA Panel outputs
    (e.g. writing validation_mpe_table.csv)
  repo: github.com/funkam/PRIMA
- name: NMR spectroscopy
  role: Quantifies metabolite levels in plasma/serum samples collected with controlled
    pre- and postcentrifugation delays
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/acs.analchem.4c04938
    title: Validating Centralized Biobanking Workflows for NMR Metabolomics Using
      the PRIMA Panel
  source_content_hash: sha256:85b04451a5774beb11a33e9f677a5d710a1b7d19d479667dcb4bfd92cc710434
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - NMR spectroscopy
  - PRIMA Panel
  - R
  - Shiny
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_3360
  - http://edamontology.org/topic_3337
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9411764705882353
n_claims: 17
---

# pre-post-centrifugation-delay-stratification

## Summary

Stratifies NMR-metabolomics blood samples by pre- vs post-centrifugation processing delay so that linear mixed models can be fitted (or applied) separately for each delay type, quantifying how each metabolite's concentration drifts with prolonged handling.

## When to use

Use when plasma/serum samples carry time-stamp-derived pre- and post-centrifugation delay metadata and you need to model or validate metabolite stability, or generate performance reports per delay type.

## When NOT to use

- Samples lack time-stamp-derived pre- or post-centrifugation delay information — the stratification and PRIMA performance report cannot be computed
- Analytes are known not to follow the linear mixed-model response (e.g. glucose in EDTA and LiHep plasma showed unexpectedly large MPE at later time points), so model-based stability claims would be misleading
- The goal is pass/fail sample QC only (Bruker BiobankQC provides that basic output); PRIMA stratification is for exploring delay effects and delay-specific modeling

## Inputs

- Blood-sample metadata table with SPREC classification and pre-/post-centrifugation time stamps (for PRIMA performance reports, a data table with pre-analytical information is uploaded)
- NMR-spectroscopy-quantified metabolite concentration table (e.g. plasma/serum profiles from EDTA/LiHep tubes) measured at time 0 and delayed time points
- Validation-cohort measurements (n = 10 precentrifugation, n = 7 postcentrifugation in the article) for MPE assessment

## Outputs

- Per-metabolite stability time points (minutes) stratified by pre- vs post-centrifugation delay
- Lollipop-plot / table summaries of delay effects split by the two delay categories
- PRIMA performance reports (.HTML or .csv) highlighting minor and major concentration changes per metabolite over a user-selected delay timeframe
- Linear mixed models per metabolite per delay stratum with validation MPE values at 2, 4, 6, and 8 h

## How to apply

From sample metadata (SPREC-classified tube types plus collection/centrifugation/aliquoting time stamps) compute the two delay categories: pre-centrifugation time (collection to centrifugation) and post-centrifugation time (centrifugation to NMR measurement/aliquoting); the PRIMA Panel's Tools tab provides this calculation from date+time stamps. Split the NMR-quantified metabolite table into the two delay strata, since the PRIMA Data tab reports stability time-points in minutes split by pre- and post-centrifugation delay. For each stratum, fit linear mixed models predicting concentration at delayed time points from the time-0 measurement, then validate predictions in an independent cohort by computing mean percentage error (MPE) per metabolite and time point (2, 4, 6, 8 h), flagging results as stable when MPE is well below 10%. The 20%-change 'stability time point' marks when a metabolite's concentration has drifted enough to matter for QC decisions.

## Related tools

- **PRIMA Panel** (Open-source R Shiny tool that stratifies stability time-points by pre-/post-centrifugation delay, computes delay times from date+time stamps, and generates performance reports from uploaded pre-analytical tables) — github.com/funkam/PRIMA
- **Shiny** (Framework hosting the PRIMA Panel app (also permanently hosted on Shinyapps.io) for interactive exploration and report download) — github.com/funkam/PRIMA
- **R** (Implementation language for the linear mixed models and PRIMA Panel outputs (e.g. writing validation_mpe_table.csv)) — github.com/funkam/PRIMA
- **NMR spectroscopy** (Quantifies metabolite levels in plasma/serum samples collected with controlled pre- and postcentrifugation delays)

## Examples

```
Rscript -e "PRIMA::generate_performance_report(pre_delay=120, post_delay=240, threshold_minor=5, threshold_major=20, output='report.html')"
```

## Evaluation signals

- Validation MPE is well below 10% at the 2- and 4-h time points for most metabolites (exception: postcentrifugation acetone in EDTA plasma)
- Glucose in EDTA and LiHep plasma shows unexpectedly large MPE at late (6 and 8 h) time points, matching the reported pattern
- Stability time points are reported in minutes and sorted according to SPREC classification, split into pre- and post-centrifugation strata
- Performance report tables highlight a minor and a major change for each metabolite over the chosen timeframe, with adjustable % thresholds and colors
- Reproduced linear mixed models predict concentration at delayed time points from the time-0 measurement; predictions should be applied separately per delay category

## Limitations

- Glucose in EDTA and LiHep plasma does not follow the linear response of the models, giving large MPE at later time points
- No explicit version information or changelog is provided for the PRIMA Panel
- The panel is a data-exploration/QC tool for peripheral-blood plasma/serum; it is not a general-purpose metabolomics workflow for other sample types
- Current Tools tab is limited to calculating pre- and post-centrifugation times from date+time stamps

## Evidence

- [readme] The data is split according the two different delays (pre- and post-centrifugation: "The data is split according the two different delays (pre- and post-centrifugation"
- [abstract] Blood samples, collected in various tube types, were subjected to controlled pre- and postcentrifugation delays: "Blood samples, collected in various tube types, were subjected to controlled pre- and postcentrifugation delays"
- [abstract] fitted in linear mixed models used to predict changes in metabolite concentrations over time: "fitted in linear mixed models used to predict changes in metabolite concentrations over time"
- [results] Validation on a separate cohort (n = 10 precentrifugation, n = 7 postcentrifugation) showed mean percentage error well below 10% for most parameters: [evidence span withheld — anchor status: not_found]
- [readme] The 'Single' tab allows the user to set a pre-centrifugation time and a post-centrifugation using a Slider.: "The 'Single' tab allows the user to set a pre-centrifugation time and a post-centrifugation using a Slider."
