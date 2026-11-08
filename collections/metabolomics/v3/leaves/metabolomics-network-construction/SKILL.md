---
name: metabolomics-network-construction
description: Use when you have a normalized metabolite concentration table (e.g. ~90
  small molecules quantified by 1H-NMR across cell populations) and need a sparse
  network of conditional dependencies among metabolite features.
when_to_use_negative:
- You need cell-type-specific sub-networks rather than a full-dataset network — the
  article runs GLASSO on the full dataset agnostic of cell type, then overlays PCA
  scores via the MetaboLINK/pcaglasso package
- Input is raw NMR spectra rather than a quantified metabolite concentration table
  — peaks must first be identified (Chenomx 800 MHz library, 380 metabolites) and
  normalized
edam_operation: http://edamontology.org/operation_3984
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0203
tools:
- name: scikit-learn
  role: Provides the sklearn.covariance.GraphLassoCV implementation used to fit GLASSO
    with cross-validated alpha (250 iterations)
- name: MetaboLINK (PCA-GLASSO algorithm)
  role: Pipeline within which GLASSO networks are overlaid with PCA scores to identify
    significant sub-networks
  repo: https://github.com/jlichtarge/pcaglasso
- name: matplotlib
  role: Plotting of the GLASSO-derived network graphs
- name: pcaglasso
  role: Python package to visualize networks; GLASSO graphs are overlaid with PCA
    scores to identify significant sub-networks
  repo: https://github.com/jlichtarge/pcaglasso
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.3389/fnins.2024.1520982
    title: MetaboLINK is a novel algorithm for unveiling cell-specific metabolic pathways
      in longitudinal datasets
  source_content_hash: sha256:ff95e8428355067a58cceb3f3cda260a3c4609774a64e876868b3af047645b53
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/jlichtarge/pcaglasso
  tools:
  - MetaboLINK (PCA-GLASSO algorithm)
  - Python
  - matplotlib
  - pcaglasso
  - scikit-learn
  edam_operation: http://edamontology.org/operation_3984
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7857142857142857
n_claims: 14
---

# metabolomics-network-construction

## Summary

Builds a sparse metabolite co-dependence network by fitting GLASSO to an NMR-derived metabolite concentration table, yielding a precision (inverse covariance) matrix whose non-zero off-diagonal entries define partial-correlation edges between metabolites.

## When to use

Use when you have a normalized metabolite concentration table (e.g. ~90 small molecules quantified by 1H-NMR across cell populations) and need a sparse network of conditional dependencies among metabolite features.

## When NOT to use

- You need cell-type-specific sub-networks rather than a full-dataset network — the article runs GLASSO on the full dataset agnostic of cell type, then overlays PCA scores via the MetaboLINK/pcaglasso package
- Input is raw NMR spectra rather than a quantified metabolite concentration table — peaks must first be identified (Chenomx 800 MHz library, 380 metabolites) and normalized

## Inputs

- Metabolite concentration table from 1H-NMR spectroscopy (e.g. Chenomx-identified peaks, ~90 metabolites x cell populations), mean- and stddev-normalized

## Outputs

- Sparse GLASSO precision (inverse covariance) matrix saved as glasso_precision_matrix.csv
- Edge list of partial correlations (positive and negative) between metabolite pairs

## How to apply

Load the metabolite concentration table (n = 90 small molecules quantified across the five neural cell populations) and apply the mean- and stddev-normalization used in the MetaboLINK pipeline. Fit GLASSO to the full dataset, agnostic of cell type, using sklearn.covariance.GraphLassoCV with default cross-validation settings and 250 iterations — cross-validation selected alpha = 0.56 in the published analysis. Extract the sparse precision matrix in which L1 regularization zeroes out metabolite pairs lacking conditional dependence, then derive an edge list of partial correlations from the precision matrix, retaining both positive and negative partial correlations as edges and leaving unconnected features out of the graph. Save the precision matrix and edge list (e.g. glasso_precision_matrix.csv). Rationale: GLASSO's L1 penalty yields a sparse, interpretable network that MetaboLINK subsequently overlays with PCA scores to identify significant sub-networks.

## Related tools

- **scikit-learn** (Provides the sklearn.covariance.GraphLassoCV implementation used to fit GLASSO with cross-validated alpha (250 iterations))
- **MetaboLINK (PCA-GLASSO algorithm)** (Pipeline within which GLASSO networks are overlaid with PCA scores to identify significant sub-networks) — https://github.com/jlichtarge/pcaglasso
- **matplotlib** (Plotting of the GLASSO-derived network graphs)
- **pcaglasso** (Python package to visualize networks; GLASSO graphs are overlaid with PCA scores to identify significant sub-networks) — https://github.com/jlichtarge/pcaglasso

## Examples

```
from sklearn.covariance import GraphLassoCV; model = GraphLassoCV(max_iter=250); model.fit(X_normalized); precision = model.precision_; alpha = model.alpha_
```

## Evaluation signals

- Confirm the fitted alpha equals the cross-validated value of 0.56 reported in the methods
- Precision matrix is sparse: many off-diagonal entries are exactly zero due to L1 regularization
- Edge list contains both positive and negative partial-correlation values, as stated in the methods
- Features with no edges are absent (removed) from the resulting graph
- Input table matches the ~90-metabolite, mean/stddev-normalized scale used by the MetaboLINK pipeline

## Limitations

- No software versions or implementation details for GLASSO are stated beyond the scikit-learn GraphLassoCV class and 250 iterations, hindering exact reproduction
- No accession or availability statement for the metabolomics dataset (hESC, hNPC, neuron samples), so inputs must be regenerated or obtained separately
- The resulting network is dataset-wide and agnostic of cell type; cell-type attribution requires the complementary PCA overlay step of MetaboLINK

## Evidence

- [methods] GLASSO is run with an alpha selected by cross-validation via the sklearn.covariance.GraphLassoCV implementation: "GLASSO is run with an alpha selected by cross-validation via the sklearn.covariance.GraphLassoCV implementation"
- [methods] We use the PCA and GLASSO implementations from scikit-learn,1 found in the sklearn.covariance packages, respectively.: "We use the PCA and GLASSO implementations from scikit-learn,1 found in the sklearn.covariance packages, respectively."
- [intro] Our method seamlessly combines principal component analysis (PCA) loadings with the graphical lasso (GLASSO) to reveal subnetworks in metabolomics datasets: "Our method seamlessly combines principal component analysis (PCA) loadings with the graphical lasso (GLASSO) to reveal subnetworks in metabolomics datasets"
- [readme] A python package to visualize networks. Graphs generated by GLASSO are overlaid with PCA scores to identify significant sub-networks.: "Graphs generated by GLASSO are overlaid with PCA scores to identify significant sub-networks."
- [methods] PCA normalization detail: "We ran PCA over the mean- and stddev-normalized dataset to produce 10 components, of which we use the first three."
