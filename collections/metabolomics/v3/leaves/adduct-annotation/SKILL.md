---
name: adduct-annotation
description: Use when converting raw LC-MS/MS mzML files into a processed peak table
  and each detected feature needs a putative adduct annotation before downstream filtering,
  normalization, or annotation with metID.
when_to_use_negative:
- The input is already a finished feature table with adduct annotations (e.g. downstream
  analysis like differential expression, sPLS, or rCCA on MultiABLER output).
- Analyzing proteomic data — MultiABLER 'currently does not provide functions to analyze
  proteomic data'; integration is only planned for future development.
- Data manipulation stages such as kNN imputation, mean normalization, or quantile
  normalization, which are explicitly not implemented in MultiABLER.
edam_operation: http://edamontology.org/operation_3632
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0154
tools:
- name: CAMERA
  role: Performs the annotate step that assigns putative adducts to grouped XCMS features
    within the runXCMS wrapper.
- name: xcms
  role: Upstream peak detection, alignment, and grouping whose grouped peak list is
    the input to CAMERA annotation.
- name: MultiABLER
  role: R pipeline that encapsulates the runXCMS workflow, including CAMERA-based
    adduct annotation, before LipidFinder/metID/EigenMS downstream steps.
  repo: https://github.com/holab-hku/MultiABLER
- name: R
  role: Runtime environment in which MultiABLER and its dependencies (xcms, CAMERA,
    devtools-installed package) are executed.
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.isci.2023.106881
    title: Integrative processing of untargeted metabolomic and lipidomic data using
      MultiABLER
  source_content_hash: sha256:05e68e3c9304725cd5c9e74cd61204a2d94c824ddbb3b19bb8c47a6aaed18403
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/holab-hku/MultiABLER
  repo_url: https://github.com/holab-hku/MultiABLER
  tools:
  - CAMERA
  - MultiABLER
  - R
  - xcms
  edam_operation: http://edamontology.org/operation_3632
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# adduct-annotation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Annotate detected LC-MS/MS chromatographic features with putative adduct/charge assignments using the CAMERA package as part of the MultiABLER XCMS pre-processing stage, producing an adduct-labeled feature table.

## When to use

Use when converting raw LC-MS/MS mzML files into a processed peak table and each detected feature needs a putative adduct annotation before downstream filtering, normalization, or annotation with metID.

## When NOT to use

- The input is already a finished feature table with adduct annotations (e.g. downstream analysis like differential expression, sPLS, or rCCA on MultiABLER output).
- Analyzing proteomic data — MultiABLER 'currently does not provide functions to analyze proteomic data'; integration is only planned for future development.
- Data manipulation stages such as kNN imputation, mean normalization, or quantile normalization, which are explicitly not implemented in MultiABLER.

## Inputs

- Grouped XCMS peak list (post centWave detection, Obiwarp alignment, groupDensity grouping) from raw mzML files
- mz and retention-time metadata for each chromatographic feature

## Outputs

- Adduct-annotated, aligned feature table
- feature_table.csv containing one row per grouped feature with a non-empty adduct annotation column

## How to apply

After peak detection (centWave), retention-time alignment (Obiwarp), and cross-sample grouping (groupDensity) in the XCMS pipeline, run CAMERA's annotate step on the grouped peak list to assign putative adducts and charges to each feature. Assemble the adduct-annotated features into a single feature table and export it as feature_table.csv, which feeds the subsequent LipidFinder filtering, metID annotation, and EigenMS normalization steps of MultiABLER. The annotation is applied per grouped feature, so grouping must complete before annotation; re-run annotation if grouping parameters change.

## Related tools

- **CAMERA** (Performs the annotate step that assigns putative adducts to grouped XCMS features within the runXCMS wrapper.)
- **xcms** (Upstream peak detection, alignment, and grouping whose grouped peak list is the input to CAMERA annotation.)
- **MultiABLER** (R pipeline that encapsulates the runXCMS workflow, including CAMERA-based adduct annotation, before LipidFinder/metID/EigenMS downstream steps.) — https://github.com/holab-hku/MultiABLER
- **R** (Runtime environment in which MultiABLER and its dependencies (xcms, CAMERA, devtools-installed package) are executed.)

## Examples

```
devtools::install_github("holab-hku/MultiABLER", dependencies = TRUE)
```

## Evaluation signals

- The produced feature table must contain aligned features with adduct annotations (non-empty adduct column).
- The feature table must have one row per grouped feature, matching the structure of the MultiABLER pre-processing output.
- The exported feature_table.csv exists and is loadable for downstream LipidFinder, metID, and normalization steps.
- Feature rows correspond to grouped (not raw, per-sample) peaks after Obiwarp alignment, confirming grouping preceded annotation.
- Annotation is regenerated consistently when centWave/Obiwarp/groupDensity parameters are re-tuned.

## Limitations

- MultiABLER's adduct/feature table pipeline does not analyze proteomic data from the Multi-ABLE extraction method; proteomic integration is planned for future development.
- Other normalization methods such as kNN imputation, mean normalization, and quantile normalization are not implemented in MultiABLER, so post-annotation correction must come from EigenMS or external tools.
- Annotation depends on correct upstream centWave parameters (ppm tolerance, peakwidth range); mis-tuned detection propagates incorrect adduct assignments.

## Evidence

- [other] peak detection with the centWave algorithm: [evidence span withheld — anchor status: not_found]
- [other] CAMERA annotate step: [evidence span withheld — anchor status: not_found]
- [other] feature_table.csv output: [evidence span withheld — anchor status: not_found]
- [other] validation of adduct annotations: [evidence span withheld — anchor status: not_found]
- [abstract] MultiABLER as unified R pipeline: "Here we present an R pipeline called MultiABLER as a unified and simple upstream processing and analysis pipeline for both metabolomics and lipidomics datasets acquired using liquid"
- [readme] GitHub installation of MultiABLER: "devtools::install_github("holab-hku/MultiABLER", dependencies = TRUE)"
