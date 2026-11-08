---
name: partial-correlation-edge-extraction
description: Use when a fitted GLASSO/GraphLassoCV precision matrix is available and
  you need an edge list of partial correlations to build a sparse metabolite network.
when_to_use_negative:
- When the goal is cell-type-specific pathway discovery rather than a full-dataset
  network — MetaboLINK applies GLASSO agnostic of cell type for this step, and stage-specific
  sub-networks come from the later PCA overlay
- When the input is an unnormalized concentration table — the mean- and stddev-normalization
  from the MetaboLINK pipeline must be applied before fitting GLASSO
- When you need raw covariance rather than conditional dependence — partial correlations
  from the precision matrix are not plain Pearson correlations
edam_operation: http://edamontology.org/operation_3435
edam_topics:
- http://edamontology.org/topic_3272
- http://edamontology.org/topic_3520
tools:
- name: scikit-learn
  role: Provides the GraphLassoCV implementation used to fit GLASSO and produce the
    sparse precision matrix from which edges are extracted
- name: pcaglasso
  role: Python package whose GLASSO-derived graphs are overlaid with PCA scores to
    identify significant sub-networks
  repo: github.com/jlichtarge/pcaGLASSO
- name: matplotlib
  role: All plotting of the extracted network graphs
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.3389/fnins.2024.1520982
    title: MetaboLINK is a novel algorithm for unveiling cell-specific metabolic pathways
      in longitudinal datasets
  source_content_hash: sha256:bfb83a12fc66ad4e051af2d090da98e5c412bd05de04b7ef4a9a0a2fd4e630f1
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Python
  - matplotlib
  - pcaglasso
  - scikit-learn
  edam_operation: http://edamontology.org/operation_3435
  edam_topics:
  - http://edamontology.org/topic_3272
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7333333333333333
n_claims: 15
---

# partial-correlation-edge-extraction

## Summary

Derives network edges from a GLASSO sparse precision (inverse covariance) matrix by converting nonzero precision entries into partial correlations, retaining both positive and negative conditional-dependence edges between metabolite features.

## When to use

Use when a fitted GLASSO/GraphLassoCV precision matrix is available and you need an edge list of partial correlations to build a sparse metabolite network.

## When NOT to use

- When the goal is cell-type-specific pathway discovery rather than a full-dataset network — MetaboLINK applies GLASSO agnostic of cell type for this step, and stage-specific sub-networks come from the later PCA overlay
- When the input is an unnormalized concentration table — the mean- and stddev-normalization from the MetaboLINK pipeline must be applied before fitting GLASSO
- When you need raw covariance rather than conditional dependence — partial correlations from the precision matrix are not plain Pearson correlations

## Inputs

- Metabolite concentration table (n = 90 small molecules quantified across the five neural cell populations), mean- and stddev-normalized
- Fitted sklearn.covariance.GraphLassoCV model with cross-validated alpha (0.56) and 250 iterations
- Sparse precision (inverse covariance) matrix

## Outputs

- Sparse precision matrix saved as glasso_precision_matrix.csv
- Signed edge list of partial correlations (positive and negative weights)
- Network retaining isolated (edgeless) features for downstream PCA-score overlay

## How to apply

First fit Graphical Lasso to the mean- and stddev-normalized metabolite concentration table (90 small molecules quantified across the five hESC-derived neural cell populations) using sklearn.covariance.GraphLassoCV with 250 iterations, which selects the L1 regularization strength by cross-validation (alpha = 0.56 in the MetaboLINK pipeline). Extract the resulting sparse precision matrix, in which L1 regularization zeroes out metabolite pairs with no conditional dependence. For each nonzero off-diagonal entry, convert the precision value to a partial correlation to obtain a signed edge weight. Build an edge list retaining both positive and negative partial correlations, and leave features with no edges in the graph (do not prune isolated nodes). Save the precision matrix and edge list (e.g. as glasso_precision_matrix.csv) so PCA scores can later be overlaid to identify significant sub-networks.

## Related tools

- **scikit-learn** (Provides the GraphLassoCV implementation used to fit GLASSO and produce the sparse precision matrix from which edges are extracted)
- **pcaglasso** (Python package whose GLASSO-derived graphs are overlaid with PCA scores to identify significant sub-networks) — github.com/jlichtarge/pcaGLASSO
- **matplotlib** (All plotting of the extracted network graphs)

## Examples

```
from sklearn.covariance import GraphLassoCV; gl = GraphLassoCV(max_iter=250); gl.fit(X_normalized); precision = gl.precision_; alpha = gl.alpha_
```

## Evaluation signals

- The cross-validated regularization strength alpha equals 0.56, as reported for the MetaboLINK GLASSO fit
- The precision matrix is sparse: many off-diagonal entries are zero (L1 regularization removed conditionally independent pairs)
- The extracted edge list contains both positive and negative partial-correlation values
- Features with no edges remain in the graph rather than being dropped
- The precision matrix and edge list are persisted as glasso_precision_matrix.csv

## Limitations

- No explicit software versions or GLASSO parameter settings beyond alpha selection and 250 iterations are documented in the article, hindering exact reproduction
- No accession or availability statement is given for the metabolomics dataset (hESC, hNPC, neuron samples), so the exact input table cannot be independently retrieved
- Edge weights are conditional (partial) correlations valid only under the cross-validated alpha; changing alpha changes which metabolite pairs are zeroed out

## Evidence

- [methods] GLASSO is run with an alpha selected by cross-validation via the sklearn.covariance.GraphLassoCV implementation: "GLASSO is run with an alpha selected by cross-validation via the sklearn.covariance.GraphLassoCV implementation"
- [methods] We ran PCA over the mean- and stddev-normalized dataset to produce 10 components, of which we use the first three.: "We ran PCA over the mean- and stddev-normalized dataset to produce 10 components, of which we use the first three."
- [readme] Graphs generated by GLASSO are overlaid with PCA scores to identify significant sub-networks.: "Graphs generated by GLASSO are overlaid with PCA scores to identify significant sub-networks."
- [results] GLASSO applied to the metabolomics data revealed a distinct network of sample features, identifying clusters and sub-clusters: [evidence span withheld — anchor status: not_found]
- [methods] All plotting is done using matplotlib,: "All plotting is done using matplotlib,"
