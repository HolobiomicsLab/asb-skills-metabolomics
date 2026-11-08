---
name: pca-score-based-mid-level-fusion
description: Use when two or more platform-specific metabolomics datasets (e.g., MS
  and NMR CSV tables) covering the same samples must be combined into one integrated
  table and raw column concatenation would duplicate correlated metabolite information
  across platforms.
when_to_use_negative:
- Only one platform dataset exists — there is nothing to fuse; use Viime's single-table
  normalization/transformation/scaling workflow instead.
- Every original metabolite concentration column must be preserved in the merged table
  — use Viime's simple (concatenative) fusion, which horizontally concatenates metabolite
  columns, rather than PCA-score reduction.
- Blocks must be variance-balanced before combining — use Viime's multi-block PCA
  fusion, which normalizes each table by its first singular value, instead of plain
  PCA-score concatenation.
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_0092
tools:
- name: Viime
  role: Web application whose dataset-merge operation dispatches to the PCA-based
    mid-level fusion algorithm and registers the merged result as a new integrated
    dataset.
  repo: https://github.com/girder/viime
- name: Scikit-learn
  role: Backend library providing the common statistical algorithms (PCA) computed
    on each normalized platform table before score concatenation.
- name: Pandas
  role: Parses the per-platform CSV tables and assembles the concatenated PCA-score
    blocks into the fused output data frame.
- name: Flask
  role: RESTful API backend that exposes the dataset merge operation to the Viime
    web client.
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.21105/joss.02410
    title: 'Viime: Visualization and Integration of Metabolomics Experiments'
  source_content_hash: sha256:e1e0022d90781bee4617dae9a0d49559dfdc688f6395a2c0b2da0e7f23405998
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/girder/viime
  tools:
  - Flask
  - Pandas
  - Scikit-learn
  - Viime
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# pca-score-based-mid-level-fusion

## Summary

Viime's mid-level data fusion: integrate two or more platform-specific metabolomics tables (e.g., MS and NMR) by concatenating the normalized PCA scores computed on each normalized dataset, producing a single fused dataset with expanded metabolome coverage.

## When to use

Use when two or more platform-specific metabolomics datasets (e.g., MS and NMR CSV tables) covering the same samples must be combined into one integrated table and raw column concatenation would duplicate correlated metabolite information across platforms.

## When NOT to use

- Only one platform dataset exists — there is nothing to fuse; use Viime's single-table normalization/transformation/scaling workflow instead.
- Every original metabolite concentration column must be preserved in the merged table — use Viime's simple (concatenative) fusion, which horizontally concatenates metabolite columns, rather than PCA-score reduction.
- Blocks must be variance-balanced before combining — use Viime's multi-block PCA fusion, which normalizes each table by its first singular value, instead of plain PCA-score concatenation.

## Inputs

- Two or more platform-specific metabolomics tables (.xlsx or .csv uploads) that have been normalized, transformed, and scaled in Viime, with a designated primary ID and retained group/factor columns
- User selection of the datasets to merge plus the mid-level (PCA) integration algorithm in the Viime dataset-merge dialog

## Outputs

- A fused integrated dataset (CSV) whose feature columns are the concatenated normalized PCA score vectors from each platform block, with sample rows aligned and group/factor columns retained
- A new dataset object registered in Viime and selectable for downstream statistical analysis and visualization

## How to apply

In Viime, complete per-platform data treatment first — normalization (reference sample, sum scaling, weight/volume), a transformation from the Log10/Log2/square root/cube root options, and a scaling choice among autoscaling, Pareto, range, vast, and level — because mid-level fusion runs PCA on each normalized table. From the data upload page, initiate a dataset merge, select the datasets to merge (uploaded as .xlsx/.csv with a designated primary ID and group/factor columns), and choose the PCA-based mid-level algorithm. The backend (Flask REST API, Pandas parsing, Scikit-learn statistics) aligns sample rows across the selected tables and concatenates the normalized PCA score vectors from each dataset as the new feature set; this differs from multi-block PCA fusion, which first normalizes each table so its first principal component has the same length (its first singular value) before combining blocks. The merged result becomes a new integrated dataset (written as CSV) whose rows are the aligned samples and whose columns are PCA scores rather than the union of metabolite concentration columns. Prefer mid-level fusion over simple (concatenative) fusion when platform blocks are large or metabolites are correlated across platforms, and over multi-block fusion when per-block singular-value normalization is not wanted.

## Related tools

- **Viime** (Web application whose dataset-merge operation dispatches to the PCA-based mid-level fusion algorithm and registers the merged result as a new integrated dataset.) — https://github.com/girder/viime
- **Scikit-learn** (Backend library providing the common statistical algorithms (PCA) computed on each normalized platform table before score concatenation.)
- **Pandas** (Parses the per-platform CSV tables and assembles the concatenated PCA-score blocks into the fused output data frame.)
- **Flask** (RESTful API backend that exposes the dataset merge operation to the Viime web client.)

## Examples

```
pipenv install; mkdir viime_sqlite; viime-cli create-tables; flask run
```

## Evaluation signals

- The merged output CSV loads cleanly as a data frame after the merge completes.
- Sample rows are aligned across blocks: row count equals the number of shared biological samples across the merged datasets, with the designated group/factor columns retained.
- Output shape matches the mid-level fusion expectation — feature columns are concatenated normalized PCA-score blocks, i.e., fewer columns than the union of raw metabolite columns that simple concatenation would produce.
- Each block's scores derive from PCA of a normalized platform table, so unnormalized raw concentration values must not appear among the fused features.
- The merged result appears in Viime as a new integrated dataset available for downstream analysis (e.g., Wilcoxon rank sum / multivariate ANOVA pages).

## Limitations

- Mid-level fusion replaces individual metabolite concentration columns with PCA scores, so per-metabolite interpretation is lost in the fused table.
- Samples must be shared across the merged platforms; rows not present in all selected datasets cannot contribute to the fused table.
- Only three integration algorithms are supported by the merge operation (simple concatenation, PCA/mid-level, multi-block PCA); other fusion strategies require custom processing.
- Fusion quality depends on prior per-platform normalization, transformation, and scaling choices; unlike multi-block fusion, plain PCA-score concatenation does not normalize each block by its first singular value.

## Evidence

- [abstract] Viime supports simple, mid-level, and multi-block data fusion across data modalities: "Viime also enables integration between different data modalities, offering simple (i.e., concatenative), mid-level, and multi-block data fusion approaches."
- [abstract] Dataset merge is initiated from the upload page with a user-chosen integration algorithm: "From the data upload page, the user may initiate a dataset merge, selecting the datasets to merge along with the algorithm to perform the integration."
- [abstract] Fused datasets expand metabolome coverage for cross-platform correlation analysis: "The resulting fused datasets offer expanded metabolome coverage, enabling an analysis of the correlated behavior of metabolites detected by different platforms."
- [abstract] Scikit-learn supplies the common statistical algorithms (e.g., PCA) in the backend: "Data processing is done by a combination of Scikit-learn for common statistical algorithms and R packages for specialized algorithms."
- [other] PCA data fusion concatenates the normalized PCA scores of each dataset: [evidence span withheld — anchor status: not_found]
- [readme] Sample CSVs can be uploaded through the running Viime client: "Once the client is running, you can use it to upload the CSV files found in the [sample data](samples/data) directory of this repository."
