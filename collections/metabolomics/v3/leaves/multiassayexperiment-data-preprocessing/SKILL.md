---
name: multiassayexperiment-data-preprocessing
description: Use when starting from a raw table of metabolite measurements plus metabolite
  (rowData) and sample (colData) metadata, and you need a clean, normalized MultiAssayExperiment
  object before differential metabolomic testing.
when_to_use_negative:
- Input is already a preprocessed MultiAssayExperiment object (annotated, imputed,
  and vsn-normalized) — re-running kNN imputation or vsn would distort the data.
- Your metabolite annotations do not come from or map to the Small Molecular Pathway
  Database (SMPDB); get_SMPDBanno expects SMPDB identifiers.
- You are past preprocessing and only need group-wise statistical testing — use diff_test
  (Student's T-Test/ANOVA with Benjamini-Hochberg correction) on the prepared object
  instead.
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: MetaboDiff
  role: Provides create_mae, get_SMPDBanno, knn_impute, remove_cluster, and normalize_met
    used to build and preprocess the MultiAssayExperiment object
  repo: http://github.com/andreasmock/MetaboDiff/
- name: MultiAssayExperiment
  role: Container class produced by create_mae that merges assay, rowData, and colData
    for downstream MetaboDiff functions
- name: Metabolon®
  role: Commercial platform that generated the exemplary tissue metabolome measurement
    table (61 prostate cancers vs 25 normal specimens) used to demonstrate the preprocessing
    workflow
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/bty344
    title: 'MetaboDiff: an R package for differential metabolomic analysis'
  source_content_hash: sha256:3cb27a14e234e00b4aa41e2ee42e26da2dbb5498fadc820f09d83c0af4c4a1db
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# multiassayexperiment-data-preprocessing

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Build and preprocess a MultiAssayExperiment object for MetaboDiff by merging the metabolite measurement table with rowData and colData, then annotating against SMPDB, imputing missing values with kNN at a 0.4 cutoff, removing outlier clusters, and applying variance stabilizing normalization (vsn).

## When to use

Use when starting from a raw table of metabolite measurements plus metabolite (rowData) and sample (colData) metadata, and you need a clean, normalized MultiAssayExperiment object before differential metabolomic testing.

## When NOT to use

- Input is already a preprocessed MultiAssayExperiment object (annotated, imputed, and vsn-normalized) — re-running kNN imputation or vsn would distort the data.
- Your metabolite annotations do not come from or map to the Small Molecular Pathway Database (SMPDB); get_SMPDBanno expects SMPDB identifiers.
- You are past preprocessing and only need group-wise statistical testing — use diff_test (Student's T-Test/ANOVA with Benjamini-Hochberg correction) on the prepared object instead.

## Inputs

- Table of metabolite measurements (assay)
- Metabolite metadata as rowData
- Sample metadata as colData (e.g. 61 prostate cancers vs 25 normal prostate specimens from the Metabolon® tissue metabolome study)
- SMPDB metabolite annotation

## Outputs

- Preprocessed MultiAssayExperiment object (annotated, imputed, outlier-corrected, vsn-normalized) ready for diff_test

## How to apply

Construct the object with `create_mae(assay, rowData, colData)`, which merges all inputs into a MultiAssayExperiment object as MetaboDiff's required container. Annotate metabolites using `get_SMPDBanno`, since metabolite annotation is retrieved from the Small Molecular Pathway Database (SMPDB). Impute missing values with `knn_impute` using k-nearest neighbor imputation at the recommended cutoff of 40% (i.e. 0.4). Inspect the outlier heatmap based on k-means clustering (the exemplary data uses k=2), then apply `remove_cluster(met, cluster=1)` to drop the identified outlier cluster. Finish with `normalize_met` using variance stabilizing normalization (vsn) so that the variance remains nearly constant across the metabolite intensities.

## Related tools

- **MetaboDiff** (Provides create_mae, get_SMPDBanno, knn_impute, remove_cluster, and normalize_met used to build and preprocess the MultiAssayExperiment object) — http://github.com/andreasmock/MetaboDiff/
- **MultiAssayExperiment** (Container class produced by create_mae that merges assay, rowData, and colData for downstream MetaboDiff functions)
- **Metabolon®** (Commercial platform that generated the exemplary tissue metabolome measurement table (61 prostate cancers vs 25 normal specimens) used to demonstrate the preprocessing workflow)

## Examples

```
met <- create_mae(assay, rowData, colData); met <- knn_impute(met, cutoff = 0.4); met <- normalize_met(met)
```

## Evaluation signals

- create_mae returns a MultiAssayExperiment object containing the assay, rowData, and colData components.
- Imputation was run with the 40% (0.4) missingness cutoff, matching the article's recommended knn_impute parameter.
- Outlier detection used k-means clustering with k=2 and the identified cluster 1 was removed via remove_cluster.
- Normalization is variance stabilizing (vsn); normalized intensities should show near-constant variance.
- The final object supports downstream diff_test, e.g. a random-gender negative control should yield no metabolite with adjusted p-value < 0.05 after Benjamini-Hochberg correction.

## Limitations

- The kNN imputation cutoff of 40% (0.4) is a recommendation; datasets with different missingness patterns may need a different cutoff or exclusion of metabolites.
- Outlier removal depends on the k-means cluster count (k=2 in the exemplary data); an inappropriate k can misidentify outlier clusters.
- MetaboDiff requires R version 4.0.2 or higher and installation via devtools install_github('andreasmock/MetaboDiff').
- The package is designed as a low-level entry point starting from a metabolite measurement table, not for raw mass-spectrometry data processing.

## Evidence
