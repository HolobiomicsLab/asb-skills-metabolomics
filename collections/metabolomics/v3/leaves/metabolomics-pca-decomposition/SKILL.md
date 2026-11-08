---
name: metabolomics-pca-decomposition
description: Use when you have a wide-format metabolomics feature table (samples as
  columns, features as rows with unique identifiers) plus a design file grouping samples,
  and you want unsupervised dimension reduction and visualization of sample structure.
when_to_use_negative:
- Input is already PCA-derived scores or a correlation matrix rather than a raw feature
  table — SVD decomposition on such inputs is meaningless.
- The analysis goal is supervised discrimination (PLS-DA, random forest, SVM); SECIMTools
  provides separate tools for those, and PCA is unsupervised.
- The dataset has not passed QC filtering (blank feature filtering, RT/CV flags) and
  imputation; PCA on an unfiltered table will be dominated by low-quality features.
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0091
tools:
- name: SECIMTools
  role: Provides the SVD-based PCA tool; consumed as the standalone Python application
    or Galaxy wrapper that performs the decomposition, scores/loadings export, and
    scatter/scree plotting.
  repo: https://github.com/SECIMTools
- name: Galaxy
  role: Wraps the SECIMTools PCA tool so it can be chained in integrated metabolomics
    workflows via the Galaxy interface.
  repo: https://github.com/SECIMTools
- name: Python
  role: Runtime language in which the PCA tool is implemented; pandas is used to load
    the wide-format table and design file before decomposition.
  repo: https://pypi.python.org/pypi/secimtools
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1186/s12859-018-2134-1
    title: 'SECIMTools: a suite of metabolomics data analysis tools'
  source_content_hash: sha256:3cdce811999001579bf90bbf788ffa912d51ad2b3336f60e95bc1846311c78e2
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Galaxy
  - Python
  - SECIMTools
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# metabolomics-pca-decomposition

## Summary

Principal component analysis of metabolomics feature tables using the SVD-based PCA tool from SECIMTools, which decomposes centered/scaled feature matrices into orthogonal PCs ordered by explained variability and emits per-sample scores, per-feature loadings, and group-colored scatter/scree plots.

## When to use

Use when you have a wide-format metabolomics feature table (samples as columns, features as rows with unique identifiers) plus a design file grouping samples, and you want unsupervised dimension reduction and visualization of sample structure.

## When NOT to use

- Input is already PCA-derived scores or a correlation matrix rather than a raw feature table — SVD decomposition on such inputs is meaningless.
- The analysis goal is supervised discrimination (PLS-DA, random forest, SVM); SECIMTools provides separate tools for those, and PCA is unsupervised.
- The dataset has not passed QC filtering (blank feature filtering, RT/CV flags) and imputation; PCA on an unfiltered table will be dominated by low-quality features.

## Inputs

- Wide-format metabolomics feature table (samples as columns, features as rows, unique feature identifiers) in CSV/TSV form
- Design file mapping sample identifiers to sample characteristics such as treatment group

## Outputs

- Per-sample scores CSV with sample identifier, design-file group, and PC1..PCn columns
- Per-feature loadings CSV keyed by feature identifier with PC weight columns
- 2D scatter plot of PC1 vs PC2 colored by group (PNG/PDF)
- 3D scatter plot of PC1-PC3 colored by group (PNG/PDF)
- Scree plot of explained variance (PNG/PDF)

## How to apply

Load the bundled example wide-format dataset with pandas, where samples are columns and metabolomics features are rows keyed by unique feature identifiers, together with the design file that relates samples to characteristics such as treatment group. Center and optionally scale the feature matrix, then decompose it with the singular value decomposition (SVD) approach that the SECIMTools PCA tool specifies. Project samples onto the principal components — which are orthogonal and ordered in descending order of explained variability — and write a per-sample scores CSV containing sample identifiers, the design-file group, and PC1..PCn columns. Extract per-feature loadings (feature weights on each PC) and write a loadings CSV keyed by feature identifier. Finally generate the tool's visual summaries: 2D and 3D scatter plots of the first three principal components with samples colored by design-file group, plus a scree plot of explained variance, saved as PNG or PDF.

## Related tools

- **SECIMTools** (Provides the SVD-based PCA tool; consumed as the standalone Python application or Galaxy wrapper that performs the decomposition, scores/loadings export, and scatter/scree plotting.) — https://github.com/SECIMTools
- **Galaxy** (Wraps the SECIMTools PCA tool so it can be chained in integrated metabolomics workflows via the Galaxy interface.) — https://github.com/SECIMTools
- **Python** (Runtime language in which the PCA tool is implemented; pandas is used to load the wide-format table and design file before decomposition.) — https://pypi.python.org/pypi/secimtools

## Examples

```
python -m secimtools.tools.secim_pca --input wide_table.csv --design design.tsv --scale --output pca_results/
```

## Evaluation signals

- The script runs end-to-end without error on the bundled example wide-format dataset plus design file.
- The scores CSV contains a sample identifier column and PC1–PC3 columns populated for every sample in the design file.
- The loadings CSV contains a feature identifier column and PC weight columns matching the number of retained components.
- A plot file (PNG or PDF) exists for the 2D/3D sample scatter and the scree plot, matching the reference plots in Additional file 2, Figures S19 and S20.
- The scree plot's explained-variance values are monotonically decreasing across PCs, confirming the PCs are ordered by descending explained variability.

## Limitations

- PCA output depends on prior quality-control and imputation steps; low-quality or missing-heavy features (e.g. those failing blank feature filtering, CV/RT flags) will distort the component structure.
- Components beyond the first three are only represented numerically (scores/loadings CSVs); the visual summaries cover PC1–PC3 only.
- Interpretation of loadings requires the wide-format table's unique feature identifiers to be consistent between input table and design file; mismatched identifiers break group coloring.

## Evidence

- [abstract] visualization techniques (hierarchical cluster heatmap, principal component analysis, modular modularity clustering): "visualization techniques (hierarchical cluster heatmap, principal component analysis, modular modularity clustering)"
- [readme] SECIMTools a set of python tools that are available both as standalone and wrapped for use in Galaxy: "The SECIMTools a set of python tools that are available both as standalone and wrapped for use in Galaxy."
- [intro] SECIMTools project aims to develop a suite of tools for processing of metabolomics data, which can be run in a standalone mode or via Galaxy: "SECIMTools project aims to develop a suite of tools for processing of metabolomics data, which can be run in a standalone mode or via Galaxy"
