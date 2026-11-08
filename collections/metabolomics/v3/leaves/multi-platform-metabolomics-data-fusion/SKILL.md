---
name: multi-platform-metabolomics-data-fusion
description: Use when two or more normalized platform-specific metabolomics tables
  (e.g., an MS table and an NMR table) cover the same biological samples and you need
  expanded metabolome coverage in one fused feature table.
when_to_use_negative:
- The datasets come from a single analytical platform — no cross-platform integration
  is needed; apply within-platform normalization/scaling instead
- Data have not yet been normalized, transformed, and scaled — Viime's intent is to
  pick up the workflow after preprocessing, and fusing raw tables will violate the
  singular-value normalization assumptions of multi-block fusion
- Sample rows cannot be aligned across platforms (different or non-unique primary
  IDs) — fused rows must correspond to the same biological samples
edam_operation: http://edamontology.org/operation_3947
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: Viime
  role: Web application providing the dataset merge operation with configurable integration
    algorithms (simple, mid-level, multi-block data fusion)
  repo: https://github.com/girder/viime
- name: Pandas
  role: Backend table handling and raw file parsing for the metabolomics datasets
    being merged
- name: Scikit-learn
  role: Implements the common statistical algorithms, including the PCA computations
    used in mid-level and multi-block fusion
- name: Flask
  role: RESTful API backend through which the merge operation is dispatched and the
    fused dataset is produced
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.21105/joss.02410
    title: 'Viime: Visualization and Integration of Metabolomics Experiments'
  source_content_hash: sha256:b9b26feb2cf7c22a53aa00435abef69fdf2e74f17ffcf317aebe4e8050011652
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/girder/viime
  tools:
  - Flask
  - Pandas
  - Scikit-learn
  - Viime
  edam_operation: http://edamontology.org/operation_3947
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# multi-platform-metabolomics-data-fusion

## Summary

Fuse metabolomics datasets measured on different analytical platforms (e.g., MS and NMR) into a single integrated dataset using Viime's three data fusion algorithms: simple concatenation, mid-level PCA-score fusion, and multi-block PCA fusion.

## When to use

Use when two or more normalized platform-specific metabolomics tables (e.g., an MS table and an NMR table) cover the same biological samples and you need expanded metabolome coverage in one fused feature table.

## When NOT to use

- The datasets come from a single analytical platform — no cross-platform integration is needed; apply within-platform normalization/scaling instead
- Data have not yet been normalized, transformed, and scaled — Viime's intent is to pick up the workflow after preprocessing, and fusing raw tables will violate the singular-value normalization assumptions of multi-block fusion
- Sample rows cannot be aligned across platforms (different or non-unique primary IDs) — fused rows must correspond to the same biological samples

## Inputs

- Two or more platform-specific metabolomics tables (CSV or .xlsx) that have been normalized, transformed, and scaled
- Per-table group/factor column designations (primary ID, group, factor columns)
- User-selected integration algorithm (simple concatenative, mid-level PCA, or multi-block PCA)

## Outputs

- A single fused dataset CSV containing the union of metabolite columns (or concatenated PCA score vectors) from all merged platforms, with sample rows aligned and platform provenance columns

## How to apply

Load each platform's normalized, transformed, and scaled metabolite table (CSV or .xlsx in Viime), then align samples so rows correspond to the same biological samples while retaining the designated group/factor columns. From the data upload page, initiate a dataset merge and select one of three algorithms: simple concatenative fusion (horizontally concatenate metabolite concentration columns), mid-level fusion (compute PCA on each normalized table and concatenate the PCA score vectors), or multi-block fusion (normalize each block so its first principal component has the same length, measured by its first singular value, then combine the blocks into a grand table). The backend implements the statistical operations with Scikit-learn (and Pandas for table handling) behind a Flask RESTful API. Choose concatenation when raw metabolite-level correlations across platforms are of interest; choose PCA-score or multi-block fusion when reduced, normalized representations are preferred. Write the fused dataset, with its expanded metabolite feature set and platform provenance columns, to an output CSV for downstream statistical analysis and visualization in Viime.

## Related tools

- **Viime** (Web application providing the dataset merge operation with configurable integration algorithms (simple, mid-level, multi-block data fusion)) — https://github.com/girder/viime
- **Pandas** (Backend table handling and raw file parsing for the metabolomics datasets being merged)
- **Scikit-learn** (Implements the common statistical algorithms, including the PCA computations used in mid-level and multi-block fusion)
- **Flask** (RESTful API backend through which the merge operation is dispatched and the fused dataset is produced)

## Examples

```
cd devops/docker && docker-compose up && yarn serve  # then, in the Viime web client, select datasets and fusion algorithm (simple / mid-level / multi-block) from the data upload page to initiate the merge
```

## Evaluation signals

- The output CSV loads cleanly as a data frame without errors
- The fused table contains the union of metabolite columns from the merged platform datasets, with sample rows aligned to the same biological samples
- The fused table's shape matches the expectation for the selected algorithm: full metabolite-column concatenation for simple fusion vs. reduced PCA-score blocks for mid-level fusion
- In multi-block fusion, each block's first principal component has equal length after division by its first singular value
- Retained group/factor columns are present in the fused dataset so downstream statistical analysis (e.g., Wilcoxon rank sum, ANOVA) can proceed

## Limitations

- Annotation is not required and Viime picks up the workflow only after preprocessing and annotation — fusion does not perform metabolite identification
- Sample alignment across datasets is a prerequisite; samples not shared across platforms cannot contribute to a fused row
- Multi-block fusion normalizes each block by its first singular value, so blocks with near-zero variance or degenerate first components may fuse poorly

## Evidence

- [abstract] Viime also enables integration between different data modalities, offering simple (i.e., concatenative), mid-level, and multi-block data fusion approaches.: "Viime also enables integration between different data modalities, offering simple (i.e., concatenative), mid-level, and multi-block data fusion approaches."
- [abstract] From the data upload page, the user may initiate a dataset merge, selecting the datasets to merge along with the algorithm to perform the integration.: "From the data upload page, the user may initiate a dataset merge, selecting the datasets to merge along with the algorithm to perform the integration."
- [abstract] The resulting fused datasets offer expanded metabolome coverage, enabling an analysis of the correlated behavior of metabolites detected by different platforms.: "The resulting fused datasets offer expanded metabolome coverage, enabling an analysis of the correlated behavior of metabolites detected by different platforms."
- [abstract] The intent of Viime package is to pick up the workflow after preprocessing and annotation and go all the way through statistical analysis and visualization.: "The intent of Viime package is to pick up the workflow after preprocessing and annotation and go all the way through statistical analysis and visualization."
- [readme] Once the client is running, you can use it to upload the CSV files found in the sample data directory of this repository.: "Once the client is running, you can use it to upload the CSV files found
in the [sample data](samples/data) directory of this repository."
