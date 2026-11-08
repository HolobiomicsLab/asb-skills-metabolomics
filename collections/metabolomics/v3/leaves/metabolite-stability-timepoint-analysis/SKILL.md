---
name: metabolite-stability-timepoint-analysis
description: Use when NMR-quantified plasma/serum metabolite levels from blood samples
  subjected to controlled pre- and postcentrifugation delays need to be modeled over
  time, or when a sample cohort's processing-delay distributions must be translated
  into predicted metabolite alterations (e.
when_to_use_negative:
- When samples were processed with negligible or tightly controlled pre- and postcentrifugation
  delays so no delay-vs-concentration variation exists to model.
- When the goal is a simple pass/fail QC call only (e.g. Bruker BiobankQC output)
  rather than quantitative per-metabolite change prediction across a delay gradient.
- When metabolite levels come from a non-NMR platform whose concentration scale and
  error structure differ from the IVDr NMR data the models were fitted on.
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_3272
- http://edamontology.org/topic_3338
tools:
- name: PRIMA Panel
  role: 'Open-source R Shiny tool that implements this analysis: explore delay effects
    interactively, set pre-/postcentrifugation times via sliders, generate performance
    reports with adjustable % thresholds and colors, and export tables as .HTML or
    .csv.'
  repo: https://github.com/funkam/PRIMA
- name: R
  role: Language in which the PRIMA Panel (linear mixed modeling and report generation)
    is implemented and deployed.
  repo: https://github.com/funkam/PRIMA
- name: Shiny
  role: Web framework hosting the PRIMA Panel, both locally and permanently on Shinyapps.io,
    for interactive exploration of stability time points and delay effects.
  repo: https://github.com/funkam/PRIMA
- name: BiobankQC
  role: Commercial Bruker tool whose basic pass/fail output the PRIMA Panel extends
    by adding processing-delay information and a biobank-user-oriented overview.
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1021/acs.analchem.4c04938
    title: Validating Centralized Biobanking Workflows for NMR Metabolomics Using
      the PRIMA Panel
  source_content_hash: sha256:6cfe3bed5efcbaf86bc9d8657868a7cc3408da8b7627ff3cde3376ac44052879
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/funkam/PRIMA
  tools:
  - BiobankQC
  - PRIMA Panel
  - R
  - Shiny
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_3272
  - http://edamontology.org/topic_3338
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# metabolite-stability-timepoint-analysis

## Summary

Quantifies, per NMR-measured metabolite in plasma/serum, how concentrations change as a function of pre- and postcentrifugation delay, using linear mixed models fitted to controlled-delay experiments and the 'stability time point' concept where a metabolite's concentration changes by 20%.

## When to use

Use when NMR-quantified plasma/serum metabolite levels from blood samples subjected to controlled pre- and postcentrifugation delays need to be modeled over time, or when a sample cohort's processing-delay distributions must be translated into predicted metabolite alterations (e.g. a performance report at a given tolerable error threshold).

## When NOT to use

- When samples were processed with negligible or tightly controlled pre- and postcentrifugation delays so no delay-vs-concentration variation exists to model.
- When the goal is a simple pass/fail QC call only (e.g. Bruker BiobankQC output) rather than quantitative per-metabolite change prediction across a delay gradient.
- When metabolite levels come from a non-NMR platform whose concentration scale and error structure differ from the IVDr NMR data the models were fitted on.

## Inputs

- NMR-quantified metabolite concentrations from plasma/serum samples with recorded pre- and postcentrifugation delay times
- Sample cohort data table with preanalytical metadata (tube type, collection and centrifugation timestamps, SPREC classification)
- Validation-cohort metabolite measurements for model robustness checks

## Outputs

- Per-metabolite linear mixed model coefficients predicting concentration change versus processing delay
- Stability time point per metabolite (delay at which concentration changes by 20%)
- Performance report: per-metabolite minor and major predicted changes at a chosen tolerable error threshold (e.g. 15%), downloadable as .csv and .HTML

## How to apply

Collect blood in various tube types and subject aliquots to controlled pre- and postcentrifugation delays, then quantify metabolite levels using NMR spectroscopy (IVDr-based serum/plasma profiling). Fit linear mixed models per metabolite to predict and quantify the concentration changes with prolonged processing delays, and define each metabolite's stability time point as the delay at which its concentration changes by 20%. Split data according to the two delay types (pre- and post-centrifugation), sort time points by SPREC classification, and model combined pre- and postcentrifugation delays to obtain a comprehensive overview of the preanalytical process. Validate the fitted models in a separate cohort to confirm robustness, then compute minor/major changes per metabolite at a chosen % threshold (e.g. 15% tolerable error) to flag outliers such as lactic acid, which is affected by even short delays.

## Related tools

- **PRIMA Panel** (Open-source R Shiny tool that implements this analysis: explore delay effects interactively, set pre-/postcentrifugation times via sliders, generate performance reports with adjustable % thresholds and colors, and export tables as .HTML or .csv.) — https://github.com/funkam/PRIMA
- **R** (Language in which the PRIMA Panel (linear mixed modeling and report generation) is implemented and deployed.) — https://github.com/funkam/PRIMA
- **Shiny** (Web framework hosting the PRIMA Panel, both locally and permanently on Shinyapps.io, for interactive exploration of stability time points and delay effects.) — https://github.com/funkam/PRIMA
- **BiobankQC** (Commercial Bruker tool whose basic pass/fail output the PRIMA Panel extends by adding processing-delay information and a biobank-user-oriented overview.)

## Examples

```
Rscript -e "shiny::runGitHub('PRIMA', 'funkam')"  # launch PRIMA Panel; upload cohort table, use Data tab to inspect SPREC-sorted stability time-points and Single tab to generate a 15%-threshold performance report as .csv/.HTML
```

## Evaluation signals

- Per-metabolite stability time points are defined consistently as the delay at which concentration changes by 20%.
- Predicted changes at a stated tolerable error threshold (e.g. 15% for the DILB cohort) match known behavior: lactic acid altered by even short delays, most metabolites stable for longer.
- Performance-report output reproduces the expected delay distributions (e.g. DILB SPREC split ~59.7% A / 4.3% A1 / 28.9% C / 6.8% E precentrifugation and 48.6% B / 34.9% D / 16.5% F postcentrifugation) and flags glycine and lactate as the altered metabolites.
- Models fitted on the training cohort remain robust when applied to the separate validation cohort.
- Exported report files (.csv/.HTML) contain per-metabolite minor and major change entries for every metabolite at the chosen threshold.

## Limitations

- No software version information is provided for the PRIMA Panel or its models, complicating reproducibility across releases.
- No explicit limitations of the models or the PRIMA Panel are discussed in the article's discussion section, so edge-case behavior (e.g. extreme delays, rare metabolites) is undocumented.
- The 'Single' performance report requires manual slider selection of pre- and postcentrifugation times, so cohort-level reports depend on user-chosen thresholds and timeframes.
- Models were developed from NMR spectroscopy data of peripheral blood (plasma/serum); applicability to other matrices or quantification platforms is not established in the provided context.

## Evidence

- [abstract] Blood samples, collected in various tube types, were subjected to controlled pre- and postcentrifugation delays; metabolite levels were quantified using NMR spectroscopy, fitted in linear mixed models used to predict changes in metabolite concentrations over time: "Blood samples, collected in various tube types, were subjected to controlled pre- and postcentrifugation delays"
- [abstract] The study introduced the concept of a 'stability time point', marking when a metabolite's concentration changes by 20%: "The study introduced the concept of a “stability time point”, marking when a metabolite’s concentration changes by 20%"
- [abstract] Some metabolites, such as lactic acid, were significantly affected by even short delays, while others remained stable for longer: "some metabolites, such as lactic acid, were significantly affected by even short delays, while others remained stable for longer"
- [readme] The Data tab shows different ways of highlighting the different stability time-points in minutes. The time-points are sorted according to their SPREC classification. The data is split according the two different delays (pre- and post-centrifugation: "The Data tab shows different ways of highlighting the different stability time-points in minutes. The time-points are sorted according to their SPREC classification."
- [readme] The 'Single' tab allows the user to set a pre-centrifugation time and a post-centrifugation using a Slider. A table is then generated that higlights a minor and a major change for each metabolite in that given timeframe. The colors, as well as the % threshholds, can be adjusted. The table can be downloaded as a .HTML report or the table directly as .csv (or other formats).: "The 'Single' tab allows the user to set a pre-centrifugation time and a post-centrifugation using a Slider. A table is then generated that higlights a minor and a major change for each metabolite in"
- [intro] These data were then used to generate linear mixed models that allow for prediction and quantification of the changes each metabolic parameter undergoes with prolonged processing delays: "These data were then used to generate linear mixed models that allow for prediction and quantification of the changes each metabolic parameter undergoes with prolonged processing delays"
- [discussion] a combination of pre- and postcentrifugation delays was modeled to obtain a comprehensive overview of preanalytical process for metabolomics samples; A validation cohort was used to investigate the robustness of these models: "a combination of pre- and postcentrifugation delays was modeled to obtain a comprehensive overview of preanalytical process for metabolomics samples"
