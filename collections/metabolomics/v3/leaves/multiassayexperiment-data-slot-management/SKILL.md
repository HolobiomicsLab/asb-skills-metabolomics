---
name: multiassayexperiment-data-slot-management
description: Use when building or evolving a MetaboDiff MultiAssayExperiment object
  and you need to track raw, knn-imputed, and variance-stabilized assay slots side-by-side
  for QC or downstream differential analysis.
when_to_use_negative:
- Input is still a raw metabolite measurement table that has not yet been merged via
  create_mae into a MultiAssayExperiment container
- The analysis goal is the downstream metabolic correlation network exploration rather
  than preprocessing and slot-level quality control of the measurement data
- The dataset has already undergone variance stabilizing normalization and imputation,
  so re-running vsn on the 'norm' slots would double-transform the measurements
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3361
tools:
- name: MetaboDiff
  role: Provides create_mae, knn_impute, remove_cluster, normalize_met, and quality_plot
    for building and slot-staging the MultiAssayExperiment object
  repo: http://github.com/andreasmock/MetaboDiff/
- name: MultiAssayExperiment
  role: Container class whose assay slots (raw, imputed, norm, norm_imputed) hold
    the staged measurement data
- name: R
  role: Runtime environment for MetaboDiff; requires R version 4.0.2 or higher
- name: Metabolon®
  role: Commercial metabolomics service whose measurement tables are the typical source
    data merged into the MultiAssayExperiment object
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/bty344
    title: 'MetaboDiff: an R package for differential metabolomic analysis'
  source_content_hash: sha256:d4d1e38cfdc5c4c46acc74fc8f91a51ede4ebd48a34a95994c6de4a24088c844
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/andreasmock/MetaboDiff
  tools:
  - MetaboDiff
  - Metabolon®
  - MultiAssayExperiment
  - R
  - vsn
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3361
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7857142857142857
n_claims: 14
---

# multiassayexperiment-data-slot-management

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Manages the staged assay slots (raw, imputed, norm, norm_imputed) of a MultiAssayExperiment object as metabolomic data progresses through the MetaboDiff preprocessing workflow, so each transformation remains traceable and comparable during quality control.

## When to use

Use when building or evolving a MetaboDiff MultiAssayExperiment object and you need to track raw, knn-imputed, and variance-stabilized assay slots side-by-side for QC or downstream differential analysis.

## When NOT to use

- Input is still a raw metabolite measurement table that has not yet been merged via create_mae into a MultiAssayExperiment container
- The analysis goal is the downstream metabolic correlation network exploration rather than preprocessing and slot-level quality control of the measurement data
- The dataset has already undergone variance stabilizing normalization and imputation, so re-running vsn on the 'norm' slots would double-transform the measurements

## Inputs

- Metabolite measurement table from a commercial vendor such as Metabolon®
- Metabolite annotation table (annotations may be retrieved from SMPDB)
- MultiAssayExperiment object output of create_mae
- MultiAssayExperiment object after knn_impute (cutoff 0.4) and remove_cluster

## Outputs

- MultiAssayExperiment object with added 'imputed' assay slot after knn_impute
- MultiAssayExperiment object with added 'norm' and 'norm_imputed' assay slots after normalize_met
- quality_plot figure showing per-sample boxplots of raw, imputed, normalized, and imputed+normalized measurements

## How to apply

Start by merging metabolite measurements and metabolite annotation into a MultiAssayExperiment object with MetaboDiff's create_mae function, which merges all objects into a single container. Apply knn_impute with the recommended 40% (0.4) missingness cutoff, which adds the imputed assay slot without discarding raw measurements. Remove outlier clusters identified by k-means clustering (e.g. k=2) via remove_cluster on the imputed data. Then run normalize_met, which applies variance stabilizing normalization (vsn) to ensure the variance remains nearly constant across the measured spectrum, appending the 'norm' and 'norm_imputed' slots to the object. Use quality_plot (e.g. with group_factor 'tumor_normal' and label_colors ('darkseagreen','dodgerblue')) to render boxplots of raw, imputed, normalized, and imputed+normalized measurements per sample, confirming that slot-level transformations behaved as intended before differential testing.

## Related tools

- **MetaboDiff** (Provides create_mae, knn_impute, remove_cluster, normalize_met, and quality_plot for building and slot-staging the MultiAssayExperiment object) — http://github.com/andreasmock/MetaboDiff/
- **MultiAssayExperiment** (Container class whose assay slots (raw, imputed, norm, norm_imputed) hold the staged measurement data)
- **R** (Runtime environment for MetaboDiff; requires R version 4.0.2 or higher)
- **Metabolon®** (Commercial metabolomics service whose measurement tables are the typical source data merged into the MultiAssayExperiment object)

## Examples

```
mae <- create_mae(metabolite_data, metabolite_annotation); mae <- knn_impute(mae, cutoff = 0.4); mae <- normalize_met(mae) # adds 'norm' and 'norm_imputed' slots
```

## Evaluation signals

- After normalize_met, the MultiAssayExperiment object contains the 'norm' and 'norm_imputed' assay slots in addition to the raw and imputed slots
- In the imputed-and-normalized panel of quality_plot, per-sample distributions show near-constant variance with no outlier sample dominating the measurement spectrum
- The distribution of measurements is comparable across the study set for the normalized and imputed data
- Missing values were imputed with knn_impute using the recommended cutoff of 40% (0.4) before normalization slots were added
- Raw measurements remain accessible in the object so raw, imputed, and normalized slots can be compared in the quality plot

## Limitations

- Normalization via vsn assumes variance should remain nearly constant over the measured spectrum; samples violating this assumption will stand out in quality_plot and may need outlier-cluster removal first
- Missingness above the 40% (0.4) cutoff is not imputed, so slot contents depend on prior filtering decisions
- The article notes no changelog ('No changelog found'), so slot naming and function behavior should be verified against the current package version (requires R 4.0.2 or higher)

## Evidence
