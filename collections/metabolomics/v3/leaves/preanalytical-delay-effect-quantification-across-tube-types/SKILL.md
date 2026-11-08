---
name: preanalytical-delay-effect-quantification-across-tube-types
description: Use when blood samples collected in EDTA/LiHep/serum tubes carry recorded
  pre- and postcentrifugation delay times and you need to predict metabolite concentration
  changes or compute 20% stability time points per tube type.
when_to_use_negative:
- Samples are not peripheral blood plasma or serum in EDTA, LiHep, or serum tubes
  (the PRIMA Panel targets plasma/serum processing delays, not other specimen types)
- You only need a pass/fail quality check of NMR spectra without modeling processing-delay
  effects (the panel was developed to go beyond basic pass/fail QC output)
- Delays are not recorded or derivable from date+time stamps, so no pre- or postcentrifugation
  times exist to model
edam_operation: http://edamontology.org/operation_3659
tools:
- name: PRIMA Panel
  role: Open-source R Shiny tool hosting the fitted delay models; used to explore
    stability time points interactively, generate performance reports from uploaded
    pre-analytical data, and calculate pre-/postcentrifugation times from date+time
    stamps
  repo: https://github.com/funkam/PRIMA
- name: R
  role: Language in which the linear mixed models are fitted and the PRIMA Panel is
    implemented and deployed locally
  repo: https://github.com/funkam/PRIMA
- name: Shiny
  role: Framework behind the PRIMA Panel web application, permanently hosted on Shinyapps.io
    and deployable locally
  repo: https://github.com/funkam/PRIMA
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/acs.analchem.4c04938
    title: Validating Centralized Biobanking Workflows for NMR Metabolomics Using
      the PRIMA Panel
  source_content_hash: sha256:fb24a2a8b681abf53934e65a2eae98d07ea0ff679909444a8fe9aad1aba493d0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/funkam/PRIMA
  tools:
  - PRIMA
  - PRIMA Panel
  - R
  - Shiny
  edam_operation: http://edamontology.org/operation_3659
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# preanalytical-delay-effect-quantification-across-tube-types

## Summary

Quantifies how NMR-quantified metabolic parameters in EDTA plasma, LiHep plasma, and serum change under pre- and postcentrifugation delays by fitting linear mixed models and deriving each parameter's 20% 'stability time point' (the modeled delay at which concentration shifts by 20%).

## When to use

Use when blood samples collected in EDTA/LiHep/serum tubes carry recorded pre- and postcentrifugation delay times and you need to predict metabolite concentration changes or compute 20% stability time points per tube type.

## When NOT to use

- Samples are not peripheral blood plasma or serum in EDTA, LiHep, or serum tubes (the PRIMA Panel targets plasma/serum processing delays, not other specimen types)
- You only need a pass/fail quality check of NMR spectra without modeling processing-delay effects (the panel was developed to go beyond basic pass/fail QC output)
- Delays are not recorded or derivable from date+time stamps, so no pre- or postcentrifugation times exist to model

## Inputs

- Delay-experiment data tables from the PRIMA repository (per metabolic parameter: tube type, pre-/postcentrifugation delay times, donor identifiers, cell counts)
- Sample-cohort table with pre-analytical information (pre- and postcentrifugation time stamps) for performance reports
- NMR spectroscopy metabolite quantification results from peripheral blood plasma/serum samples

## Outputs

- Recomputed 20% stability time points per metabolic parameter, tube type, and delay type (e.g. results_table.csv)
- Performance reports highlighting minor/major changes per metabolite within user-defined pre-/postcentrifugation timeframes (.HTML or .csv)
- Interactive lollipop plots and SPREC-sorted stability time-point tables in minutes

## How to apply

Download the delay-experiment data tables from the PRIMA GitHub repository (github.com/funkam/PRIMA) and structure them per metabolic parameter with tube type (EDTA, LiHep, serum), pre- and postcentrifugation delay times, donor identifiers, and cell counts. For each metabolic parameter, fit a linear mixed model predicting concentration change over prolonged processing delays, with cell count as random slope and donor as random intercept. Extract, per parameter and delay type (pre- vs postcentrifugation), the modeled time at which concentration changes by 20% — the stability time point — and split results by tube type. Combine the fitted models into the PRIMA Panel R Shiny tool to generate performance reports for a sample cohort by uploading a table of pre-analytical information; the 'Single' tab lets you set pre- and postcentrifugation times via sliders and highlights minor/major changes per metabolite with adjustable % thresholds. Recomputed values are checked against reported anchors, e.g. lactic acid precentrifugation 0.41 h in EDTA plasma (≈25 min) and postcentrifugation 0.81 h (EDTA), 1.63 h (LiHep), 4.3 h (serum).

## Related tools

- **PRIMA Panel** (Open-source R Shiny tool hosting the fitted delay models; used to explore stability time points interactively, generate performance reports from uploaded pre-analytical data, and calculate pre-/postcentrifugation times from date+time stamps) — https://github.com/funkam/PRIMA
- **R** (Language in which the linear mixed models are fitted and the PRIMA Panel is implemented and deployed locally) — https://github.com/funkam/PRIMA
- **Shiny** (Framework behind the PRIMA Panel web application, permanently hosted on Shinyapps.io and deployable locally) — https://github.com/funkam/PRIMA

## Evaluation signals

- Recomputed lactic acid precentrifugation stability time point in EDTA plasma reproduces the reported 0.41 h anchor (≈25 min delay for a 20% change) within the table's reported precision
- Recomputed lactic acid postcentrifugation stability time points match the anchors 0.81 h (EDTA), 1.63 h (LiHep), and 4.3 h (serum)
- Recomputed lactic acid/glucose ratio change falls in the reported 21–32 min range
- Results table (results_table.csv) covers every metabolic parameter × tube type (EDTA, LiHep, serum) × delay type (pre-/postcentrifugation) combination
- Models consistently use cell count as random slope and donor as random intercept, and were validated in a separate cohort (Biobank Dresden) for robustness

## Limitations

- No changelog or version information is provided for the PRIMA Panel repository, complicating reproducibility pinning
- Model robustness was checked with a validation cohort, but no explicit limitations of the models or the PRIMA Panel are discussed in the source text
- Stability time points are specific to NMR-spectroscopy-quantified metabolites and the controlled room-temperature delay experiments; short-delay-sensitive metabolites like lactic acid show very tight tolerances (e.g. 0.41 h) that may not generalize to other workflows

## Evidence

- [abstract] stability time point definition: "The study introduced the concept of a “stability time point”, marking when a metabolite’s concentration changes by 20%"
- [intro] linear mixed models: "These data were then used to generate linear mixed models that allow for prediction and quantification of the changes each metabolic parameter undergoes with prolonged processing delays"
- [abstract] model purpose: "fitted in linear mixed models used to predict changes in metabolite concentrations over time"
- [readme] PRIMA data tab: "The Data tab shows different ways of highlighting the different stability time-points in minutes. The time-points are sorted according to their SPREC classification."
- [readme] performance reports: "The 'Single' tab allows the user to set a pre-centrifugation time and a post-centrifugation using a Slider. A table is then generated that higlights a minor and a major change for each metabolite"
- [other] anchor validation values: [evidence span withheld — anchor status: not_found]
- [intro] scope beyond QC: "PRIMA was developed to enhance this functionality, offering a broader range of information on processing delays, and a user-friendly overview tailored for biobank users."
