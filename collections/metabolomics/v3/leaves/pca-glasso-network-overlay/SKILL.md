---
name: pca-glasso-network-overlay
description: Use when high-dimensional longitudinal metabolomics data (e.g., NMR-derived
  metabolite concentrations across developmental stages) must be decomposed into transition-specific
  covariance subnetworks linking metabolites to specific cell-type transitions.
when_to_use_negative:
- Input is raw spectral data rather than a normalized metabolite concentration table
  — NMR pre-processing with TopSpin/Chenomx peak identification must happen first
- The analysis goal is general network inference without a longitudinal or transition-based
  dimension of interest — a plain GLASSO network alone suffices
- Sample size or feature count is too small for sparse precision-matrix estimation
  with cross-validation, making GraphLassoCV unstable
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0091
tools:
- name: MetaboLINK (PCA-GLASSO algorithm)
  role: Overall algorithm combining PCA loadings with GLASSO to reveal transition-specific
    subnetworks
  repo: https://github.com/jlichtarge/pcaGLASSO
- name: pcaglasso
  role: Python package that overlays PCA scores on GLASSO-generated graphs to identify
    significant sub-networks
  repo: https://github.com/jlichtarge/pcaGLASSO
- name: scikit-learn
  role: Provides the PCA implementation (wrapped by 'pca_bundle') and sklearn.covariance.GraphLassoCV
    for GLASSO with cross-validated alpha
- name: matplotlib
  role: Visualization of the GLASSO network with overlaid PCA scores
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.3389/fnins.2024.1520982
    title: MetaboLINK is a novel algorithm for unveiling cell-specific metabolic pathways
      in longitudinal datasets
  source_content_hash: sha256:602d325265a737468f2e5bcf1bcc1c6a5d37ba2ba9d754dc9111d1f257b9f270
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/jlichtarge/pcaGLASSO
  tools:
  - MetaboLINK (PCA-GLASSO algorithm)
  - Python
  - matplotlib
  - pcaglasso
  - scikit-learn
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8461538461538461
n_claims: 13
---

# pca-glasso-network-overlay

## Summary

MetaboLINK-style overlay of PCA loadings/scores onto a GLASSO-estimated sparse covariance network to identify transition-specific metabolite subnetworks in longitudinal metabolomics data. GLASSO supplies the covariance edges while PCA scores color nodes by their contribution to a cell-type transition.

## When to use

Use when high-dimensional longitudinal metabolomics data (e.g., NMR-derived metabolite concentrations across developmental stages) must be decomposed into transition-specific covariance subnetworks linking metabolites to specific cell-type transitions.

## When NOT to use

- Input is raw spectral data rather than a normalized metabolite concentration table — NMR pre-processing with TopSpin/Chenomx peak identification must happen first
- The analysis goal is general network inference without a longitudinal or transition-based dimension of interest — a plain GLASSO network alone suffices
- Sample size or feature count is too small for sparse precision-matrix estimation with cross-validation, making GraphLassoCV unstable

## Inputs

- Mean- and stddev-normalized metabolite concentration table (e.g., 90 metabolites across five neural cell populations from 1H-NMR, e.g., Bruker Avance 800 MHz with cryoprobe)
- Sequential cell-type pair definitions for transition-wise PCA (e.g., hESC-EB, EB-Rosettes, Rosettes-hNPC, hNPC-Neurons)
- GLASSO regularization parameters: alpha = 0.56 with 250 iterations for the full dataset; alpha = 0.9 for transition-pair networks

## Outputs

- Full GLASSO sparse covariance network over metabolites (positive and negative partial correlations)
- PCA component scores/loadings and explained variance (10 components, first three used) plus pairwise transition scores per metabolite
- Transition-specific labelled subnetwork files containing nodes, edges, and PCA-score annotations for each of the four cell transitions

## How to apply

Load the metabolite concentration table (e.g., 90 metabolites across five neural cell populations) with pandas and apply mean- and stddev-normalization. Run PCA via the 'pca_bundle' class wrapping scikit-learn, producing 10 components of which the first three are used; additionally run PCA over each sequential cell-type pair (hESC-EB, EB-Rosettes, Rosettes-hNPC, hNPC-Neurons) to obtain a binary classification associating metabolites with one of the two cell types. Run GLASSO on the full dataset via sklearn.covariance.GraphLassoCV with alpha = 0.56 and 250 iterations (cell-type agnostic), and with alpha = 0.9 on each truncated cell-transition pair, yielding a sparse covariance network of positive and negative correlations. Overlay the per-feature PCA scores onto the GLASSO network edges and visualize with matplotlib so metabolite scores are placed within the context of feature covariance relationships, then extract transition-specific subnetworks (e.g., the hESC-to-EB subnetwork containing glutamate and GABA) and write them as labelled subnetwork files with nodes, edges, and PCA-score annotations.

## Related tools

- **MetaboLINK (PCA-GLASSO algorithm)** (Overall algorithm combining PCA loadings with GLASSO to reveal transition-specific subnetworks) — https://github.com/jlichtarge/pcaGLASSO
- **pcaglasso** (Python package that overlays PCA scores on GLASSO-generated graphs to identify significant sub-networks) — https://github.com/jlichtarge/pcaGLASSO
- **scikit-learn** (Provides the PCA implementation (wrapped by 'pca_bundle') and sklearn.covariance.GraphLassoCV for GLASSO with cross-validated alpha)
- **matplotlib** (Visualization of the GLASSO network with overlaid PCA scores)

## Examples

```
from sklearn.covariance import GraphLassoCV; gl = GraphLassoCV(alphas=[0.56], max_iter=250); gl.fit(X_normalized); from pcaglasso import pca_bundle; pca_bundle(n_components=10).fit(X_normalized)
```

## Evaluation signals

- Labelled subnetwork files exist for each of the four cell transitions (hESC-EB, EB-Rosettes, Rosettes-hNPC, hNPC-Neurons)
- Each subnetwork file contains GLASSO edges annotated with PCA scores for its transition
- Full-dataset GLASSO run uses alpha = 0.56 with 250 iterations; transition-pair runs use alpha = 0.9
- PCA produces 10 components with the first three used, on the mean- and stddev-normalized dataset
- Qualitative reproduction of the reported transition-specific metabolome structure, e.g., the hESC-to-EB subnetwork containing glutamate, GABA, and related metabolites

## Limitations

- The pathway database(s) used for validation are not named in the article, hindering reproducibility of the validation step
- No software versions or explicit MetaboLINK version/parameter statement beyond the alpha values is given
- No accession or availability statement for the analyzed metabolomics dataset (hESC, hNPC, neuron samples) is provided

## Evidence

- [methods] PCA over normalized data and GLASSO parameters: "We ran PCA over the mean- and stddev-normalized dataset to produce 10 components, of which we use the first three."
- [methods] GLASSO implementation via sklearn.covariance: "We use the PCA and GLASSO implementations from scikit-learn,1 found in the sklearn.covariance packages, respectively."
- [intro] Method rationale: combining PCA loadings with GLASSO to reveal subnetworks: "Our method seamlessly combines principal component analysis (PCA) loadings with the graphical lasso (GLASSO) to reveal subnetworks in metabolomics datasets"
- [readme] pcaglasso package purpose from README: "Graphs generated by GLASSO are overlaid with PCA scores to identify significant sub-networks."
- [methods] pca_bundle helper class: "The class ‘pca_bundle’ runs PCA and allows the easy visualization of PCA-related graphs, showing loadings, scores, and explained variance."
