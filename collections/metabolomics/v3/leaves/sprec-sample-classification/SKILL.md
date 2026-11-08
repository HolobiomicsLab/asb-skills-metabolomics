---
name: sprec-sample-classification
description: Use when a sample cohort table contains collection, centrifugation, and
  freezing date-time stamps and each sample must receive a SPREC classification (letters
  2 and 6) reflecting its pre- and postcentrifugation processing delays.
when_to_use_negative:
- The cohort lacks collection, centrifugation, or freezing timestamps — the Tools
  tab requires date+time stamps to compute the delay differences
- Samples are not peripheral blood plasma/serum — the PRIMA Panel explicitly targets
  'samples of peripheral blood (plasma / serum)' and its SPREC codes and stability
  models are NMR-metabolomics specific
- The goal is a pass/fail QC verdict only (e.g., Bruker BiobankQC-style output) rather
  than SPREC-coded delay classification and concentration-change prediction
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_3518
tools:
- name: PRIMA Panel
  role: Provides the Tools tab that calculates pre- and post-centrifugation times
    from timestamp differences and the Data tab where stability time-points are sorted
    by SPREC classification
  repo: github.com/funkam/PRIMA
- name: R
  role: Implementation language of the open-source PRIMA Panel Shiny tool used for
    delay computation and SPREC-based sorting
  repo: github.com/funkam/PRIMA
- name: Shiny
  role: Interactive web framework hosting the PRIMA Panel (permanently hosted on Shinyapps.io)
    through which timestamp tables are uploaded and SPREC-classified
  repo: github.com/funkam/PRIMA
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/acs.analchem.4c04938
    title: Validating Centralized Biobanking Workflows for NMR Metabolomics Using
      the PRIMA Panel
  source_content_hash: sha256:6fc774796883ae34a05326bac70d3dc3f13d9c6acc3c7ddff5e416464716a2c5
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - PRIMA Panel
  - R
  - Shiny
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_3518
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# sprec-sample-classification

## Summary

Assigns SPREC codes to each plasma/serum sample in a cohort by computing pre- and postcentrifugation delays from collection, centrifugation, and freezing date-time stamps, enabling standardized sorting of NMR-metabolomics stability time-points in the PRIMA Panel.

## When to use

Use when a sample cohort table contains collection, centrifugation, and freezing date-time stamps and each sample must receive a SPREC classification (letters 2 and 6) reflecting its pre- and postcentrifugation processing delays.

## When NOT to use

- The cohort lacks collection, centrifugation, or freezing timestamps — the Tools tab requires date+time stamps to compute the delay differences
- Samples are not peripheral blood plasma/serum — the PRIMA Panel explicitly targets 'samples of peripheral blood (plasma / serum)' and its SPREC codes and stability models are NMR-metabolomics specific
- The goal is a pass/fail QC verdict only (e.g., Bruker BiobankQC-style output) rather than SPREC-coded delay classification and concentration-change prediction

## Inputs

- Table of sample date+time stamps: collection time, centrifugation time, and freezing time per sample (uploaded to the PRIMA Panel)
- Tube-type / container metadata for each blood sample (plasma or serum), as SPREC codes encode sample type and handling

## Outputs

- Per-sample SPREC classification codes (letters 2 and 6) derived from the computed pre- and post-centrifugation delays
- Pre- and post-centrifugation delay values (in minutes) per sample, used for sorting stability time-points and driving the linear mixed-model predictions

## How to apply

Load the cohort's timestamp table (collection, centrifugation, freezing date+time columns) into the PRIMA Panel's Tools tab, which calculates the pre-centrifugation delay as the difference between collection and centrifugation timestamps and the post-centrifugation delay as the difference between centrifugation and freezing timestamps. Based on the computed delays, assign the SPREC classification to each sample at code positions 2 and 6, which encode the preanalytical handling steps. In the PRIMA Panel Data tab, stability time-points in minutes are then sorted according to their SPREC classification, and the data is split according to the two different delays (pre- and post-centrifugation). The classified delays feed the deposited pre- and postcentrifugation linear mixed models that predict per-parameter concentration changes, so correct SPREC assignment directly determines which stability time-points and model predictions apply to each sample.

## Related tools

- **PRIMA Panel** (Provides the Tools tab that calculates pre- and post-centrifugation times from timestamp differences and the Data tab where stability time-points are sorted by SPREC classification) — github.com/funkam/PRIMA
- **R** (Implementation language of the open-source PRIMA Panel Shiny tool used for delay computation and SPREC-based sorting) — github.com/funkam/PRIMA
- **Shiny** (Interactive web framework hosting the PRIMA Panel (permanently hosted on Shinyapps.io) through which timestamp tables are uploaded and SPREC-classified) — github.com/funkam/PRIMA

## Evaluation signals

- Pre-centrifugation delay equals the centrifugation-minus-collection timestamp difference, and post-centrifugation delay equals the freezing-minus-centrifugation difference, for every sample row
- SPREC letters 2 and 6 are populated for each sample and are consistent with the computed pre-/post-centrifugation delay magnitudes
- Stability time-points in the PRIMA Panel Data tab are correctly ordered when sorted according to their SPREC classification
- Downstream linear mixed-model predictions at the classified delays reproduce the paper's 'stability time point' concept (a metabolite concentration change of 20%)
- Uploaded timestamp table round-trips: the same input regenerates identical delay values and SPREC assignments across sessions of the hosted app

## Limitations

- No changelog or version information is provided for the PRIMA Panel repository, so SPREC logic may drift between deployments (hosted Shinyapps.io instance vs. local)
- SPREC codes only capture positions 2 and 6 here (pre-/postcentrifugation handling); other SPREC positions (e.g., tube type) depend on metadata completeness of the uploaded table
- The article notes the PRIMA Panel was developed to extend BiobankQC's basic pass/fail output, but no explicit limitations of the classification logic are discussed in the paper

## Evidence

- [readme] The time-points are sorted according to their SPREC classification.: "The time-points are sorted according to their SPREC classification. In addition, there is an alternative way of presenting the date in form of lollipop plots."
- [readme] Tools-tab delay calculation from date+time stamps: "An additional tab for Tools is available. Currently it consists of a tool for calculating the pre- and post-centrifugation times from the differences of date+time stamps."
- [other] Assign SPREC classification (letters 2 and 6) to each sample based on the computed delays.: [evidence span withheld — anchor status: not_found]
- [abstract] Controlled pre- and postcentrifugation delays applied to blood samples in various tube types: "Blood samples, collected in various tube types, were subjected to controlled pre- and postcentrifugation delays"
- [discussion] Combination of pre- and postcentrifugation delays modeled for a comprehensive overview: "a combination of pre- and postcentrifugation delays was modeled to obtain a comprehensive overview of preanalytical process for metabolomics samples"
