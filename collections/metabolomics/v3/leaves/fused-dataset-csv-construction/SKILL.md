---
name: fused-dataset-csv-construction
description: Use when two or more normalized platform-specific metabolomics tables
  covering the same biological samples must be combined into one dataset with expanded
  metabolome coverage, or when choosing among concatenative, mid-level, and multi-block
  fusion strategies.
when_to_use_negative:
- Only one platform's dataset is available — there is nothing to fuse, so continue
  with normalization, imputation, or statistical analysis instead.
- The input tables are raw, un-normalized, or un-annotated — Viime's intent is to
  pick up the workflow after preprocessing and annotation, so complete those steps
  before merging.
- The goal is per-platform statistical testing (e.g., Wilcoxon rank sum or ANOVA with
  Tukey post-hoc) rather than cross-platform integration.
edam_operation: http://edamontology.org/operation_0336
edam_topics:
- http://edamontology.org/topic_3922
- http://edamontology.org/topic_3474
tools:
- name: Viime
  role: Hosts the dataset merge operation from the data upload page, dispatching to
    the chosen integration algorithm and registering the fused dataset for analysis
  repo: https://github.com/girder/viime
- name: Pandas
  role: Parses the input CSV tables, performs sample alignment and the horizontal
    concatenation of metabolite concentration columns, and writes the output CSV
  repo: https://github.com/girder/viime
- name: Scikit-learn
  role: Computes the per-table PCA used for mid-level PCA-score fusion and the singular-value
    normalization in multi-block fusion
  repo: https://github.com/girder/viime
- name: Flask
  role: Exposes the merge/integration step as a RESTful API endpoint in the Viime
    processing backend
  repo: https://github.com/girder/viime
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.21105/joss.02410
    title: 'Viime: Visualization and Integration of Metabolomics Experiments'
  source_content_hash: sha256:89cc561cd34ba2a4ae4c6a97bb04af86f857113dc44c0b01064fe4f35f1948f0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/girder/viime
  tools:
  - Flask
  - Pandas
  - Scikit-learn
  - Viime
  edam_operation: http://edamontology.org/operation_0336
  edam_topics:
  - http://edamontology.org/topic_3922
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# fused-dataset-csv-construction

## Summary

Constructs a single integrated (fused) metabolomics dataset CSV from two or more platform-specific tables (e.g., MS and NMR) by sample alignment plus one of Viime's three data fusion algorithms: simple concatenation, mid-level PCA-score fusion, or multi-block PCA fusion.

## When to use

Use when two or more normalized platform-specific metabolomics tables covering the same biological samples must be combined into one dataset with expanded metabolome coverage, or when choosing among concatenative, mid-level, and multi-block fusion strategies.

## When NOT to use

- Only one platform's dataset is available — there is nothing to fuse, so continue with normalization, imputation, or statistical analysis instead.
- The input tables are raw, un-normalized, or un-annotated — Viime's intent is to pick up the workflow after preprocessing and annotation, so complete those steps before merging.
- The goal is per-platform statistical testing (e.g., Wilcoxon rank sum or ANOVA with Tukey post-hoc) rather than cross-platform integration.

## Inputs

- two or more normalized, transformed, and scaled platform-specific metabolomics tables (CSV, one per platform, e.g. MS and NMR)
- sample alignment keys and designated group/factor columns per dataset
- user-selected integration algorithm (simple concatenative, mid-level PCA, or multi-block PCA)

## Outputs

- a single fused/integrated dataset CSV containing the union of metabolite columns from the merged platforms with sample rows aligned
- platform provenance columns in the output CSV
- a new integrated dataset registered in Viime for downstream statistical analysis and visualization

## How to apply

Load each normalized, transformed, and scaled platform table from its stored CSV representation, then align rows so they correspond to the same biological samples while retaining the designated group/factor columns. Select the integration algorithm: for simple (concatenative) fusion, horizontally concatenate the metabolite concentration columns of each table; for mid-level fusion, compute PCA on each normalized table and concatenate the PCA score vectors; for multi-block fusion, first normalize each block by dividing by its first singular value so the first principal component has equal length across blocks, then combine into a grand table. The fused result becomes a new integrated dataset available for downstream analysis in Viime, which is served by a Flask RESTful backend that uses Pandas for CSV handling and Scikit-learn for PCA. Write the fused table — with its expanded metabolite feature set and platform provenance columns — to an output CSV that loads cleanly as a data frame.

## Related tools

- **Viime** (Hosts the dataset merge operation from the data upload page, dispatching to the chosen integration algorithm and registering the fused dataset for analysis) — https://github.com/girder/viime
- **Pandas** (Parses the input CSV tables, performs sample alignment and the horizontal concatenation of metabolite concentration columns, and writes the output CSV) — https://github.com/girder/viime
- **Scikit-learn** (Computes the per-table PCA used for mid-level PCA-score fusion and the singular-value normalization in multi-block fusion) — https://github.com/girder/viime
- **Flask** (Exposes the merge/integration step as a RESTful API endpoint in the Viime processing backend) — https://github.com/girder/viime

## Examples

```
cd devops/docker && docker-compose up
```

## Evaluation signals

- The output CSV loads cleanly as a data frame without parsing errors.
- The output contains the union of metabolite columns from the merged platform datasets with sample rows aligned.
- The output shape matches the selected fusion algorithm: full metabolite-column width for raw concatenation vs. reduced PCA-score blocks for mid-level or multi-block fusion.
- Designated group/factor columns are retained and rows still correspond to the same biological samples after alignment.
- The fused dataset appears as a new integrated dataset selectable for downstream analysis in Viime.

## Limitations

- Sample alignment requires that the same biological samples are present across the selected datasets; misaligned samples are a failure mode of the row-alignment step.
- Algorithm choice changes output dimensionality (raw concatenation keeps the full feature set, PCA-based fusion reduces blocks to score vectors), so downstream analyses are sensitive to the selected fusion method.
- Viime's multi-block normalization balances blocks by first singular value, which assumes each block's first component length is a meaningful scale reference; blocks with dominated or degenerate first components may fuse poorly.
- The R-side CSV processing functions exposed via OpenCPU are described in the README as work-in-progress backend code.

## Evidence

- [abstract] Three fusion algorithms: "Viime also enables integration between different data modalities, offering simple (i.e., concatenative), mid-level, and multi-block data fusion approaches."
- [abstract] Merge entry point: "From the data upload page, the user may initiate a dataset merge, selecting the datasets to merge along with the algorithm to perform the integration."
- [abstract] Fused dataset purpose: "The resulting fused datasets offer expanded metabolome coverage, enabling an analysis of the correlated behavior of metabolites detected by different platforms."
- [readme] CSV processing backend: "This installation includes a custom R package called [viime](devops/viime), which contains all functions that are exposed to the API server for CSV file processing."
