---
name: metabolite-stability-thresholding
description: Use when a biobank sample cohort has pre- and postcentrifugation timestamps
  and you need to determine, per metabolic parameter and tube type, the delay at which
  a 20% concentration change occurs and flag minor vs major changes.
when_to_use_negative:
- The input is already NMR raw spectra rather than a pre-analytical timestamp/model
  table — the PRIMA Panel operates on delay information and precomputed models, not
  raw NMR files.
- You need a simple pass/fail sample QC call without per-parameter stability assessment
  — Bruker's BiobankQC alone suffices and the PRIMA Panel's stability thresholding
  adds detail beyond that scope.
- The metabolic parameters are not among those quantified by NMR spectroscopy with
  IVDr methods; stability time points were only established for those parameters.
edam_operation: http://edamontology.org/operation_0564
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_0091
tools:
- name: PRIMA Panel
  role: R Shiny application that hosts the stability time points, the Data tab for
    SPREC-sorted delay highlighting, and the Performance Reports 'Single' tab for
    thresholded minor/major change tables
  repo: https://github.com/funkam/PRIMA
- name: R
  role: Language in which the PRIMA Panel and its linear mixed models are implemented
    and deployed
  repo: https://github.com/funkam/PRIMA
- name: Shiny
  role: Interactive web framework used to expose the slider-based threshold adjustment
    and report export (.HTML/.csv)
  repo: https://github.com/funkam/PRIMA
- name: NMR spectroscopy
  role: Method used to quantify metabolite levels underlying the fitted stability
    models
provenance:
  source_task_ids:
  - task_004
  - task_003
  source_papers:
  - doi: 10.1021/acs.analchem.4c04938
    title: Validating Centralized Biobanking Workflows for NMR Metabolomics Using
      the PRIMA Panel
  source_content_hash: sha256:f0b14ffa7143b1514c2d951634b151cc4b45ece1b6b1ed1db339f939c24e542e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/funkam/PRIMA
  tools:
  - NMR spectroscopy
  - PRIMA Panel
  - R
  - Shiny
  edam_operation: http://edamontology.org/operation_0564
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# metabolite-stability-thresholding

## Summary

Uses the PRIMA Panel's stability-time-point concept to flag the processing delay at which each NMR-quantified metabolic parameter in EDTA plasma, serum, or LiHep plasma changes by a 20% threshold, classifying changes as minor or major for a sample cohort.

## When to use

Use when a biobank sample cohort has pre- and postcentrifugation timestamps and you need to determine, per metabolic parameter and tube type, the delay at which a 20% concentration change occurs and flag minor vs major changes.

## When NOT to use

- The input is already NMR raw spectra rather than a pre-analytical timestamp/model table — the PRIMA Panel operates on delay information and precomputed models, not raw NMR files.
- You need a simple pass/fail sample QC call without per-parameter stability assessment — Bruker's BiobankQC alone suffices and the PRIMA Panel's stability thresholding adds detail beyond that scope.
- The metabolic parameters are not among those quantified by NMR spectroscopy with IVDr methods; stability time points were only established for those parameters.

## Inputs

- Date-time stamps of collection, centrifugation, and freezing per sample (pre-analytical information table)
- Sample tube type / SPREC classification (EDTA plasma, serum, LiHep plasma)
- Linear mixed model tables for pre- and postcentrifugation concentration changes per metabolic parameter
- Configurable % change threshold (tolerable-error threshold) for classifying minor and major changes

## Outputs

- Per-parameter stability time points in minutes, sorted by SPREC classification
- Performance report table flagging minor and major concentration changes per metabolite for a given pre-/postcentrifugation delay pair
- Count of metabolic parameters whose stability time point falls within a given window (e.g. 4 h) per sample type and delay type

## How to apply

Load the PRIMA Panel (R Shiny, hosted at https://funkam.shinyapps.io/PRIMA/) with linear mixed models fitted to the NMR spectroscopy data from the Dresden Integrated Liquid Biobank, which predict concentration changes in each metabolic parameter as a function of pre- and postcentrifugation delays. For each parameter and sample type (EDTA plasma, serum, LiHep plasma), compute the stability time point, i.e. the delay in minutes at which the modeled concentration change reaches 20%. Sort these time points by SPREC classification and split them by delay type (pre- vs postcentrifugation) as shown in the Data tab; validate against the paper's reference counts of parameters with a precentrifugation stability time point within 4 h (26 for EDTA plasma, 26 for serum, 15 for LiHep plasma; postcentrifugation 26/23/8). For cohort-level reporting, use the Performance Reports 'Single' tab: set pre- and postcentrifugation times via sliders, apply the adjustable % threshold (the tolerable-error threshold) to classify minor and major changes per metabolite, and export the colored table as .HTML or .csv. Rationale: the models combine pre- and postcentrifugation effects to give a comprehensive overview of preanalytical variation, enhancing Bruker BiobankQC's basic pass/fail output.

## Related tools

- **PRIMA Panel** (R Shiny application that hosts the stability time points, the Data tab for SPREC-sorted delay highlighting, and the Performance Reports 'Single' tab for thresholded minor/major change tables) — https://github.com/funkam/PRIMA
- **R** (Language in which the PRIMA Panel and its linear mixed models are implemented and deployed) — https://github.com/funkam/PRIMA
- **Shiny** (Interactive web framework used to expose the slider-based threshold adjustment and report export (.HTML/.csv)) — https://github.com/funkam/PRIMA
- **NMR spectroscopy** (Method used to quantify metabolite levels underlying the fitted stability models)

## Examples

```
shiny::runGitHub("funkam/PRIMA")  # open Tools tab to compute pre/post-centrifugation delays from date+time stamps, then Performance Reports for thresholded tables
```

## Evaluation signals

- Per-parameter stability time points reproduce the 20% concentration-change definition when recomputed from the deposited linear mixed models (github.com/funkam/PRIMA).
- Recomputed precentrifugation counts of parameters with a stability time point ≤ 240 min equal the published values: 26 (EDTA plasma), 26 (serum), 15 (LiHep plasma).
- Recomputed postcentrifugation counts equal 26 (EDTA plasma), 23 (serum), 8 (LiHep plasma).
- The exported performance report table colors metabolite rows consistent with the adjustable minor/major % threshold selected by the user.
- Predicted concentration changes from pre- and postcentrifugation models are applied in combination, matching the paper's comprehensive-overview approach.

## Limitations

- No explicit limitations of the models or the PRIMA Panel are discussed in the provided article sections.
- No changelog found for the repository, and no version information is provided for the PRIMA Panel.
- Stability time points are defined by a fixed 20% change threshold, which may not suit all downstream applications without re-tuning the adjustable % threshold.

## Evidence

- [abstract] The study introduced the concept of a 'stability time point', marking when a metabolite's concentration changes by 20%: "The study introduced the concept of a “stability time point”, marking when a metabolite’s concentration changes by 20%"
- [other] Precentrifugation stability counts within 4 h by sample type: [evidence span withheld — anchor status: not_found]
- [readme] Performance Reports tab behavior: "A table is then generated that higlights a minor and a major change for each metabolite in that given timeframe. The colors, as well as the % threshholds, can be adjusted."
- [readme] Data tab / SPREC sorting: "The Data tab shows different ways of highlighting the different stability time-points in minutes. The time-points are sorted according to their SPREC classification."
- [readme] Tools tab delay computation: "Currently it consists of a tool for calculating the pre- and post-centrifugation times from the differences of date+time stamps."
