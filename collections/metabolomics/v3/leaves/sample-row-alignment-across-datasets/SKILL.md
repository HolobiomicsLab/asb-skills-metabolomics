---
name: sample-row-alignment-across-datasets
description: Use when two or more normalized, transformed, and scaled metabolomics
  tables — one per analytical platform — must be merged into a single integrated dataset
  in Viime and the sample rows need to correspond to the same biological samples across
  the merged blocks.
when_to_use_negative:
- Only a single platform dataset is present — merging/alignment is unnecessary; proceed
  directly to normalization, transformation, scaling, and PCA analysis
- The workflow is still at preprocessing/annotation stage; Viime's intent is to pick
  up after preprocessing and annotation, and annotation is not required
- Samples in the platform tables do not correspond to the same biological samples
  — row alignment would force incorrect correspondences
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_3315
- http://edamontology.org/topic_0091
tools:
- name: Viime
  role: Web application whose dataset merge dialog performs sample row alignment and
    dispatches to one of the three supported integration algorithms (simple, mid-level,
    multi-block fusion)
  repo: https://github.com/girder/viime
- name: Pandas
  role: Backend library used to parse the source CSV tables and to align and concatenate
    the sample rows / metabolite columns during the merge operation
  repo: https://github.com/girder/viime
- name: Scikit-learn
  role: Computes the PCA scores on each normalized table for mid-level data fusion
    and the singular values used to normalize blocks in multi-block fusion
  repo: https://github.com/girder/viime
- name: Flask
  role: RESTful backend that exposes the merge operation and writes the fused dataset
    to an output CSV
  repo: https://github.com/girder/viime
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.21105/joss.02410
    title: 'Viime: Visualization and Integration of Metabolomics Experiments'
  source_content_hash: sha256:d1affd35d9e6a0258b915a68bcb663e124c720d3800897a47448841b4077e2b8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/girder/viime
  tools:
  - Flask
  - Pandas
  - Scikit-learn
  - Viime
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_3315
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# Sample row alignment across datasets

## Summary

Align sample rows across platform-specific metabolomics tables (e.g. MS and NMR feature tables exported as .csv/.xlsx) so that rows correspond to the same biological samples before applying Viime's simple (concatenative), mid-level (PCA score fusion), or multi-block (first-singular-value normalization) data fusion algorithms.

## When to use

Use when two or more normalized, transformed, and scaled metabolomics tables — one per analytical platform — must be merged into a single integrated dataset in Viime and the sample rows need to correspond to the same biological samples across the merged blocks.

## When NOT to use

- Only a single platform dataset is present — merging/alignment is unnecessary; proceed directly to normalization, transformation, scaling, and PCA analysis
- The workflow is still at preprocessing/annotation stage; Viime's intent is to pick up after preprocessing and annotation, and annotation is not required
- Samples in the platform tables do not correspond to the same biological samples — row alignment would force incorrect correspondences

## Inputs

- Normalized, transformed, and scaled metabolomics tables (one per analytical platform, e.g. MS and NMR), stored as CSV files
- Sample metadata columns designated as the primary ID, factor, or group in each source table
- User selection of which datasets to merge and which integration algorithm to use (simple, mid-level, or multi-block)

## Outputs

- A fused/integrated dataset CSV containing the union of metabolite columns from the merged platform datasets with sample rows aligned
- Platform provenance columns identifying the source dataset of each feature
- A new integrated dataset available for further analysis in Viime

## How to apply

From Viime's data upload page, initiate a dataset merge and select the datasets to combine along with the integration algorithm. First align the samples across the selected platform datasets so rows correspond to the same biological samples, retaining the designated group/factor columns. Then apply the chosen algorithm: for simple (concatenative) fusion, horizontally concatenate the metabolite concentration columns of each table; for mid-level fusion, compute PCA on each normalized table with Scikit-learn and concatenate the PCA score vectors; for multi-block fusion, divide each block by its first singular value so each table's first principal component has equal length before combining into a grand table. The backend (Flask + Pandas, with specialized algorithms in R via OpenCPU) writes the fused dataset, with its expanded metabolite feature set and platform provenance columns, to an output CSV.

## Related tools

- **Viime** (Web application whose dataset merge dialog performs sample row alignment and dispatches to one of the three supported integration algorithms (simple, mid-level, multi-block fusion)) — https://github.com/girder/viime
- **Pandas** (Backend library used to parse the source CSV tables and to align and concatenate the sample rows / metabolite columns during the merge operation) — https://github.com/girder/viime
- **Scikit-learn** (Computes the PCA scores on each normalized table for mid-level data fusion and the singular values used to normalize blocks in multi-block fusion) — https://github.com/girder/viime
- **Flask** (RESTful backend that exposes the merge operation and writes the fused dataset to an output CSV) — https://github.com/girder/viime

## Examples

```
viime-cli create-tables && flask run   # then in the Viime UI: Data Upload → initiate dataset merge → select datasets + integration algorithm (simple, mid-level, or multi-block)
```

## Evaluation signals

- The output CSV loads cleanly as a data frame (e.g. via Pandas read_csv) without row-index or header errors
- The output contains the union of metabolite columns from all merged platform datasets, with sample rows aligned (each row is one biological sample across all blocks)
- The output shape matches the expectation of the chosen algorithm: raw concatenation yields sum-of-columns across blocks, while PCA-based fusion yields reduced score-vector blocks (mid-level) or singular-value-normalized blocks (multi-block)
- The designated group/factor columns are preserved in the fused dataset so downstream statistical analyses (Wilcoxon rank sum, ANOVA/Tukey) remain valid
- Platform provenance is traceable — each feature column can be attributed to its originating dataset

## Limitations

- Sample correspondence is assumed from the user-selected datasets; Viime does not describe automatic cross-platform sample matching, so mislabeled primary IDs will propagate through the fused dataset
- Only the three implemented fusion approaches are supported (simple concatenative, mid-level PCA, multi-block PCA); other integration algorithms are not available
- Viime does not handle raw preprocessing or annotation — inputs must already be preprocessed and annotated (annotation itself is not required, but preprocessing is assumed done elsewhere)

## Evidence

- [abstract] Viime also enables integration between different data modalities, offering simple (i.e., concatenative), mid-level, and multi-block data fusion approaches.: "Viime also enables integration between different data modalities, offering simple (i.e., concatenative), mid-level, and multi-block data fusion approaches."
- [abstract] From the data upload page, the user may initiate a dataset merge, selecting the datasets to merge along with the algorithm to perform the integration.: "From the data upload page, the user may initiate a dataset merge, selecting the datasets to merge along with the algorithm to perform the integration."
- [abstract] Fused datasets offer expanded metabolome coverage, enabling analysis of correlated behavior of metabolites detected by different platforms.: "The resulting fused datasets offer expanded metabolome coverage, enabling an analysis of the correlated behavior of metabolites detected by different platforms."
- [abstract] The backend leverages Pandas for raw file parsing and normalization.: "The backend leverages Pandas for raw file parsing and normalization."
