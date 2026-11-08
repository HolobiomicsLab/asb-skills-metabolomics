---
name: svd-based-dimensionality-reduction
description: Use when a wide-format metabolomics feature matrix (samples as columns,
  unique feature IDs as rows) plus a design file mapping samples to groups needs ordination,
  group separation assessment, or feature-weight inspection.
when_to_use_negative:
- The input is already peak-picked and needs upstream quality-control filtering first
  — use SECIMTools QC steps (Blank Feature Filtering, CV Flags, RT Flags) before ordination
- The goal is supervised discrimination between known groups — PLS-DA or random forest
  in SECIMTools is the appropriate choice, not unsupervised PCA
- The data require a per-feature statistical test (ANOVA, t-test, Kruskal-Wallis)
  rather than sample-level ordination
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0091
tools:
- name: SECIMTools
  role: provides the PCA tool that specifies the SVD-based decomposition, scores/loadings
    CSV outputs, and the 2D/3D scatter and scree plots
  repo: https://github.com/SECIMTools
- name: Galaxy
  role: optional wrapper — SECIMTools PCA can be run via the Galaxy platform or standalone
  repo: https://github.com/SECIMTools
- name: Python
  role: SECIMTools PCA is a Python application; run standalone or via the secimtools
    pypi package
  repo: https://pypi.python.org/pypi/secimtools
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1186/s12859-018-2134-1
    title: 'SECIMTools: a suite of metabolomics data analysis tools'
  source_content_hash: sha256:ccc266c42bea8ac8573ccb060f72d0d4b030e2b502e4c4fbd0012e8f1dacbc15
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
n_claims: 11
---

# svd-based-dimensionality-reduction

## Summary

Principal component analysis of metabolomics feature tables via singular value decomposition, as implemented in the SECIMTools PCA tool: scores, loadings, and 2D/3D visual summaries colored by design-file group.

## When to use

Use when a wide-format metabolomics feature matrix (samples as columns, unique feature IDs as rows) plus a design file mapping samples to groups needs ordination, group separation assessment, or feature-weight inspection.

## When NOT to use

- The input is already peak-picked and needs upstream quality-control filtering first — use SECIMTools QC steps (Blank Feature Filtering, CV Flags, RT Flags) before ordination
- The goal is supervised discrimination between known groups — PLS-DA or random forest in SECIMTools is the appropriate choice, not unsupervised PCA
- The data require a per-feature statistical test (ANOVA, t-test, Kruskal-Wallis) rather than sample-level ordination

## Inputs

- wide-format feature table (samples as columns, metabolomics features as rows with unique feature identifiers, e.g. the bundled example dataset)
- design file relating samples to sample characteristics such as treatment group

## Outputs

- per-sample scores CSV containing sample identifier, design-file group, and PC1..PCn columns
- per-feature loadings CSV keyed by feature identifier
- 2D and 3D scatter plots of the first three principal components with samples colored by design-file group (PNG or PDF)
- scree plot of explained variance (PNG or PDF)

## How to apply

Load the wide-format feature table (samples as columns, features as rows with unique identifiers) together with the design file relating samples to characteristics such as treatment group. Center and optionally scale the feature matrix, then decompose it with singular value decomposition as the SECIMTools PCA tool specifies. Project samples onto the orthogonal principal components, which are ordered by descending explained variability, and write a per-sample scores CSV containing the sample identifier, the design-file group, and PC1..PCn columns; extract per-feature loadings and write a loadings CSV keyed by feature identifier. Generate the tool's visual summaries: 2D and 3D scatter plots of the first three principal components with samples colored by design-file group, plus a scree plot of explained variance, saved as PNG or PDF.

## Related tools

- **SECIMTools** (provides the PCA tool that specifies the SVD-based decomposition, scores/loadings CSV outputs, and the 2D/3D scatter and scree plots) — https://github.com/SECIMTools
- **Galaxy** (optional wrapper — SECIMTools PCA can be run via the Galaxy platform or standalone) — https://github.com/SECIMTools
- **Python** (SECIMTools PCA is a Python application; run standalone or via the secimtools pypi package) — https://pypi.python.org/pypi/secimtools

## Examples

```
python secimtools/tools/pca.py --input features.csv --design design.csv --output results/
```

## Evaluation signals

- Scores CSV contains the sample identifier, the design-file group, and PC1..PCn columns
- Loadings CSV contains the feature identifier and PC columns
- A plot file (PNG or PDF) exists matching the reference plots in Additional file 2, Figures S19 and S20
- Principal components are orthogonal and ordered in descending order of explained variability
- Scree plot reported, showing explained variance per component

## Limitations

- PCA is unsupervised; it reveals structure but does not test group differences — pair with PLS-DA, ANOVA, or random forest for inference
- Features should be QC-filtered (e.g. Blank Feature Filtering, CV Flags) and missing values imputed (group mean/median or KNN) before decomposition, otherwise low-quality features dominate loadings
- Explained variability ordering assumes centered (and optionally scaled) features; scaling choice changes loading weights and PC ordering

## Evidence

- [methods] PCA tool workflow: [evidence span withheld — anchor status: not_found]
- [abstract] visualization techniques: "visualization techniques (hierarchical cluster heatmap, principal component analysis, modular modularity clustering)"
- [intro] standalone or Galaxy: "which can be run in a standalone mode or via Galaxy"
- [readme] README synopsis: "The SECIMTools a set of python tools that are available both as standalone and wrapped for use in Galaxy."
- [readme] PCA availability: "principal component analysis, linear discriminant analysis, modular modularity clustering"
