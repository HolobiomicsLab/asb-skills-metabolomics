---
name: preanalytical-workflow-quality-assessment
description: Use when a sample cohort carries preanalytical metadata (collection,
  centrifugation, freezing timestamps for plasma/serum samples) and you need to assess
  which metabolic parameters changed due to processing delays, e.g., before NMR-based
  metabolomics analysis or biobank QC.
when_to_use_negative:
- Samples are not peripheral blood plasma/serum or lack date-time stamps for collection,
  centrifugation, and freezing — the delay computation requires all three timestamps.
- The analysis already has a finalized feature table and only needs downstream statistical
  testing, not preanalytical delay modeling.
- Only a pass/fail QC verdict is needed and no parameter-level change quantification
  is desired — Bruker's BiobankQC already covers that narrower scope.
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3272
- http://edamontology.org/topic_0091
tools:
- name: PRIMA Panel
  role: Open-source R Shiny tool that hosts the delay calculator (Tools tab), the
    SPREC-sorted stability data view, and the 'Single' tab performance report generator
    with adjustable thresholds
  repo: github.com/funkam/PRIMA
- name: R
  role: Language in which the PRIMA Panel is implemented; used to fit and apply the
    linear mixed models
  repo: github.com/funkam/PRIMA
- name: Shiny
  role: Web framework hosting the interactive app (also permanently hosted on Shinyapps.io
    at https://funkam.shinyapps.io/PRIMA/)
  repo: github.com/funkam/PRIMA
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/acs.analchem.4c04938
    title: Validating Centralized Biobanking Workflows for NMR Metabolomics Using
      the PRIMA Panel
  source_content_hash: sha256:37876b570606a8cc948447e9854dd9a8c6fcd17ed2fa401d131a27fbeb4dc5ab
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - PRIMA Panel
  - R
  - Shiny
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3272
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# preanalytical-workflow-quality-assessment

## Summary

Generates a PRIMA Panel performance report for a plasma/serum sample cohort by computing pre- and post-centrifugation delays from collection, centrifugation, and freezing date-time stamps, then applying linear mixed models to predict per-metabolite concentration changes. Supports SPREC-style classification of samples and filtering against a configurable percent-error threshold.

## When to use

Use when a sample cohort carries preanalytical metadata (collection, centrifugation, freezing timestamps for plasma/serum samples) and you need to assess which metabolic parameters changed due to processing delays, e.g., before NMR-based metabolomics analysis or biobank QC.

## When NOT to use

- Samples are not peripheral blood plasma/serum or lack date-time stamps for collection, centrifugation, and freezing — the delay computation requires all three timestamps.
- The analysis already has a finalized feature table and only needs downstream statistical testing, not preanalytical delay modeling.
- Only a pass/fail QC verdict is needed and no parameter-level change quantification is desired — Bruker's BiobankQC already covers that narrower scope.

## Inputs

- Data table with pre-analytical information: collection, centrifugation, and freezing date-time stamps per sample (plasma/serum)
- Pre- and post-centrifugation linear mixed model tables for each metabolic parameter
- SPREC classification codes (letters 2 and 6) for each sample
- User-adjustable % threshold defining tolerable change (e.g. 20% stability time point)

## Outputs

- Performance report table highlighting minor and major changes per metabolite within a specified timeframe
- Downloadable .HTML report or .csv table of predicted concentration changes
- Data-driven sample/parameter filtering recommendation to increase reliability of subsequent analyses
- Stability time points (in minutes) per metabolite, sorted by SPREC classification

## How to apply

1. Upload or load a data table with pre-analytical information — collection date-time, centrifugation date-time, and freezing date-time for each sample. 2. Use the PRIMA Panel's Tools tab to calculate pre-centrifugation delay (centrifugation minus collection) and post-centrifugation delay (freezing minus centrifugation) from the date+time stamp differences. 3. Classify each sample by SPREC category (letters 2 and 6) based on the computed delays, matching the Data tab's stability time-point sorting. 4. Generate a performance report via the 'Single' tab: set a pre-centrifugation and post-centrifugation time using sliders, then the panel produces a table highlighting minor and major concentration changes for each metabolite. 5. Adjust color and % thresholds interactively to define what counts as a tolerable change (the paper ties 20% change to the 'stability time point' concept). 6. Export the report as .HTML or the underlying table as .csv (or other formats). Rationale: linear mixed models fitted to pre- and post-centrifugation delay data predict the magnitude of change each metabolic parameter undergoes, enabling data-driven filtering of samples and parameters before downstream NMR metabolomics.

## Related tools

- **PRIMA Panel** (Open-source R Shiny tool that hosts the delay calculator (Tools tab), the SPREC-sorted stability data view, and the 'Single' tab performance report generator with adjustable thresholds) — github.com/funkam/PRIMA
- **R** (Language in which the PRIMA Panel is implemented; used to fit and apply the linear mixed models) — github.com/funkam/PRIMA
- **Shiny** (Web framework hosting the interactive app (also permanently hosted on Shinyapps.io at https://funkam.shinyapps.io/PRIMA/)) — github.com/funkam/PRIMA

## Examples

```
shiny::runGitHub('funkam/PRIMA')  # then open the Tools tab to compute pre/post-centrifugation delays and the 'Single' tab to generate a performance report
```

## Evaluation signals

- Computed pre-centrifugation delay equals centrifugation timestamp minus collection timestamp; post-centrifugation equals freezing minus centrifugation (no negative delays).
- Each sample is assigned a SPREC classification consistent with its computed delays, and samples sort into the same SPREC bins as the Data tab's stability time-point view.
- Predicted concentration change for a metabolite at the user's chosen delay pair matches the linear mixed model output and is flagged 'minor' or 'major' according to the user-set % threshold.
- A metabolite's predicted change reaches ~20% exactly at its documented 'stability time point'.
- Exported report is a valid .HTML file, and the table export parses as a clean .csv with one row per metabolite.

## Limitations

- No software version numbers or changelog are stated for the PRIMA Panel, so reproducibility of a specific model version is not guaranteed.
- The paper does not explicitly discuss limitations of the linear mixed models or of the PRIMA Panel itself in the reviewed sections.
- The tool is scoped to the effects of pre- and post-centrifugation delays on plasma/serum metabolic parameters; other preanalytical variables (e.g., tube type effects beyond what the models capture) are not covered by the performance report.
- Interpretation depends on user-chosen % thresholds, which can be adjusted and thus are not universally standardized.

## Evidence

- [readme] Performance report computation from preanalytical timestamps: "The 'Single' tab allows the user to set a pre-centrifugation time and a post-centrifugation using a Slider. A table is then generated that higlights a minor and a major change for each metabolite in"
- [readme] Tools tab delay calculation: "An additional tab for Tools is available. Currently it consists of a tool for calculating the pre- and post-centrifugation times from the differences of date+time stamps."
- [abstract] Stability time point concept: [evidence span withheld — anchor status: not_found]
- [abstract] Model fitting rationale: "fitted in linear mixed models used to predict changes in metabolite concentrations over time"
- [readme] Output download format: "The table can be downloaded as a .HTML report or the table directly as .csv (or other formats)."
- [discussion] Modeling combined delays: "a combination of pre- and postcentrifugation delays was modeled to obtain a comprehensive overview of preanalytical process for metabolomics samples"
- [intro] Enhancing BiobankQC: "PRIMA was developed to enhance this functionality, offering a broader range of information on processing delays, and a user-friendly overview tailored for biobank users."
