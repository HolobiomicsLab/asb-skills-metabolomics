---
name: scree-plot-generation
description: Use when an SVD-based PCA has been fit on a centered (and optionally
  scaled) wide-format metabolomics feature matrix and you need to summarize explained
  variability per principal component as a plot for output alongside PCA scores and
  loadings.
when_to_use_negative:
- The input is not a PCA decomposition (e.g., raw peak tables from data preprocessing)
  — filtering tools like BFF, CV Flags, or RT Flags apply instead
- The analysis goal is supervised (PLS-DA, random forest, SVM, LASSO/Elastic Net)
  rather than unsupervised PCA variance summary
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: SECIMTools
  role: Provides the PCA tool whose SVD-based decomposition supplies the explained-variance
    values plotted in the scree plot
  repo: https://github.com/SECIMTools
- name: Python
  role: Standalone implementation language for loading the wide-format dataset, running
    SVD-based PCA, and rendering the scree plot as PNG/PDF
  repo: https://pypi.python.org/pypi/secimtools
- name: Galaxy
  role: Web-based platform in which the SECIMTools PCA tool (and its scree plot output)
    can be wrapped and run in integrated workflows
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1186/s12859-018-2134-1
    title: 'SECIMTools: a suite of metabolomics data analysis tools'
  source_content_hash: sha256:5309ff06223e128f7ff52f30138149f802ded2dcc87ea93877d14daa7f8f1b5b
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
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# scree-plot-generation

## Summary

Generate a scree plot of explained variance from an SVD-based principal component analysis of metabolomics feature data, as implemented in the SECIMTools PCA tool, to visualize how much variability each ordered principal component captures.

## When to use

Use when an SVD-based PCA has been fit on a centered (and optionally scaled) wide-format metabolomics feature matrix and you need to summarize explained variability per principal component as a plot for output alongside PCA scores and loadings.

## When NOT to use

- The input is not a PCA decomposition (e.g., raw peak tables from data preprocessing) — filtering tools like BFF, CV Flags, or RT Flags apply instead
- The analysis goal is supervised (PLS-DA, random forest, SVM, LASSO/Elastic Net) rather than unsupervised PCA variance summary

## Inputs

- Wide-format metabolomics feature matrix (samples as columns, features as rows with unique feature identifiers), loaded with pandas in Python
- Design file relating samples to sample characteristics such as treatment group
- Fitted SVD-based PCA decomposition (explained variance per principal component)

## Outputs

- Scree plot of explained variance (or explained variance ratio) per principal component, saved as PNG or PDF
- Optionally paired 2D and 3D scatter plots of the first three principal components colored by design-file group

## How to apply

After centering and (optionally) scaling the feature matrix (samples as columns, metabolomics features as rows with unique feature identifiers), decompose it with the singular value decomposition (SVD) approach specified by the SECIMTools PCA tool. Project samples onto the orthogonal principal components, which are ordered in descending order of explained variability. Extract the per-component explained variance (e.g., singular values squared divided by n-1, or the explained variance ratio) and plot it against component number (PC1, PC2, ... PCn) as the scree plot. Save the scree plot as PNG or PDF alongside the 2D/3D PC1-PC3 scatter plots, per-sample scores CSV, and per-feature loadings CSV. The scree plot supports the decision point of how many PCs capture meaningful variability before downstream statistical tools such as PLS-DA, ANOVA, or random forest are applied within the SECIMTools suite.

## Related tools

- **SECIMTools** (Provides the PCA tool whose SVD-based decomposition supplies the explained-variance values plotted in the scree plot) — https://github.com/SECIMTools
- **Python** (Standalone implementation language for loading the wide-format dataset, running SVD-based PCA, and rendering the scree plot as PNG/PDF) — https://pypi.python.org/pypi/secimtools
- **Galaxy** (Web-based platform in which the SECIMTools PCA tool (and its scree plot output) can be wrapped and run in integrated workflows)

## Examples

```
from sklearn.decomposition import PCA; ev = PCA(svd_solver='full').fit(X_centered).explained_variance_ratio_; plt.plot(range(1, len(ev)+1), ev, 'o-'); plt.savefig('scree.png')
```

## Evaluation signals

- The script runs end-to-end without errors and a scree plot file (PNG or PDF) exists on disk
- The scree plot's explained-variance values are monotonically non-increasing from PC1 to PCn, consistent with PCs ordered in descending order of explained variability
- The number of plotted components matches the rank of the centered feature matrix (at most min(n_samples-1, n_features))
- The companion scores CSV contains sample identifier, design-file group, and PC1-PC3 columns, and the loadings CSV contains the feature identifier and PC columns
- Plot content matches the reference plots in Additional file 2, Figures S19 and S20

## Limitations

- No version number for SECIMTools or Galaxy is given in the discussion, so reproducibility of the exact plot styling may require pinning a package version
- PCA is unsupervised; the scree plot describes total explained variability and does not by itself indicate group separation by treatment group
- Features with missing values should be handled first (e.g., via the SECIMTools Imputation tool, group mean/median or KNN) since SVD requires a complete matrix

## Evidence

- [abstract] visualization techniques such as hierarchical cluster heatmap, PCA, and modularity clustering: [evidence span withheld — anchor status: not_found]
- [readme] readme visualization list: "visualization techniques (hierarchical cluster heatmap, principal component analysis, linear discriminant analysis, modular modularity clustering)"
- [readme] standalone or Galaxy: "SECIMTools project aims to develop a suite of tools for processing of metabolomics data, which can be run in a standalone mode or via Galaxy Genomics Framework."
