---
name: cell-transition-metabolite-association
description: Use when you have longitudinal/high-dimensional metabolomics data (e.g.,
  1H-NMR concentration tables) spanning sequential cell states and need to extract
  transition-specific metabolite subnetworks rather than global correlations.
when_to_use_negative:
- The goal is pathway-level metabolite set enrichment rather than transition-specific
  covariance subnetworks (the paper instead cross-checks against KEGG, HMDB, and BMRB
  for validation)
- The dataset has no ordered longitudinal structure, so there are no sequential cell-state
  pairs over which to define binary PCA classifications
- The input is already a validated biomarker list or pathway assignment, not a raw
  metabolite-by-sample concentration matrix
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3272
- http://edamontology.org/topic_0153
tools:
- name: MetaboLINK (PCA-GLASSO algorithm)
  role: 'The overarching algorithm: combines PCA loadings with Graphical Lasso to
    identify significant transition-specific subnetworks'
  repo: github.com/jlichtarge/pcaGLASSO
- name: pcaglasso
  role: Python package that overlays PCA scores onto GLASSO-generated graphs to identify
    significant sub-networks
  repo: github.com/jlichtarge/pcaGLASSO
- name: scikit-learn
  role: Provides the PCA and GraphLassoCV (GLASSO with cross-validated alpha) implementations
    from sklearn.covariance
- name: matplotlib
  role: Used for all plotting, including visualizing PCA-score-overlaid GLASSO networks
- name: pca_bundle
  role: Class that runs PCA and allows easy visualization of PCA-related graphs (loadings,
    scores, explained variance)
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.3389/fnins.2024.1520982
    title: MetaboLINK is a novel algorithm for unveiling cell-specific metabolic pathways
      in longitudinal datasets
  source_content_hash: sha256:2019a4ab750a2c5ce6d4c9d7ef750ddbdd43dbff4226aa6d1ec86923916d1c9f
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MetaboLINK (PCA-GLASSO algorithm)
  - Python
  - matplotlib
  - pca_bundle
  - pcaglasso
  - scikit-learn
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3272
  - http://edamontology.org/topic_0153
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8461538461538461
n_claims: 13
---

# cell-transition-metabolite-association

## Summary

Associates individual metabolites with specific cell-type transitions by pairing per-transition binary PCA classification with GLASSO covariance networks, so each metabolite is placed within the covariance-network context of a developmental transition (e.g., hESC-to-EB).

## When to use

Use when you have longitudinal/high-dimensional metabolomics data (e.g., 1H-NMR concentration tables) spanning sequential cell states and need to extract transition-specific metabolite subnetworks rather than global correlations.

## When NOT to use

- The goal is pathway-level metabolite set enrichment rather than transition-specific covariance subnetworks (the paper instead cross-checks against KEGG, HMDB, and BMRB for validation)
- The dataset has no ordered longitudinal structure, so there are no sequential cell-state pairs over which to define binary PCA classifications
- The input is already a validated biomarker list or pathway assignment, not a raw metabolite-by-sample concentration matrix

## Inputs

- Mean- and stddev-normalized metabolite concentration table (e.g., 90 metabolites from 1H-NMR spectra of five neural cell populations: hESC, EB, Rosettes, hNPC, Neurons)
- Ordered cell-type transition labels defining the sequential pairs (hESC-EB, EB-Rosettes, Rosettes-hNPC, hNPC-Neurons)
- GLASSO regularization settings (alpha = 0.56, 250 iterations for the full network; alpha = 0.9 for per-transition networks)

## Outputs

- Labelled transition-specific subnetwork files containing nodes, edges, and PCA-score annotations for each of the four cell transitions
- GLASSO covariance networks (full dataset and per-transition pairs) with edges annotated by PCA loading scores
- Matplotlib visualizations of the PCA-score-overlaid GLASSO networks

## How to apply

Load the metabolite concentration table (n = 90 metabolites across five neural cell populations) and apply mean- and stddev-normalization. Using the 'pca_bundle' class wrapping scikit-learn, run PCA on the normalized dataset (10 components, first three used) and run PCA over each sequential cell-type pair (hESC-EB, EB-Rosettes, Rosettes-hNPC, hNPC-Neurons) to frame a binary classification associating individual metabolites with one of the two cell types. Run GLASSO cell-type-agnostically on the full dataset via sklearn.covariance.GraphLassoCV with alpha = 0.56 and 250 iterations, and on each truncated transition pair with alpha = 0.9, yielding sparse covariance networks capturing positive and negative correlations. Overlay the per-feature PCA scores onto the GLASSO network edges and visualize with matplotlib, then extract the transition-specific subnetworks (e.g., the reported hESC-to-EB subnetwork containing glutamate, GABA, and related metabolites) as labelled subnetwork files with nodes, edges, and PCA-score annotations. The rationale is that GLASSO supplies the covariance structure while PCA loadings supply the transition context, allowing metabolites to be judged significant within a transition rather than globally.

## Related tools

- **MetaboLINK (PCA-GLASSO algorithm)** (The overarching algorithm: combines PCA loadings with Graphical Lasso to identify significant transition-specific subnetworks) — github.com/jlichtarge/pcaGLASSO
- **pcaglasso** (Python package that overlays PCA scores onto GLASSO-generated graphs to identify significant sub-networks) — github.com/jlichtarge/pcaGLASSO
- **scikit-learn** (Provides the PCA and GraphLassoCV (GLASSO with cross-validated alpha) implementations from sklearn.covariance)
- **matplotlib** (Used for all plotting, including visualizing PCA-score-overlaid GLASSO networks)
- **pca_bundle** (Class that runs PCA and allows easy visualization of PCA-related graphs (loadings, scores, explained variance))

## Examples

```
from sklearn.covariance import GraphLassoCV; gl = GraphLassoCV(); gl.fit(X_normalized)  # full dataset (alpha~0.56, 250 iters), then repeat per transition pair with alpha=0.9 and overlay pca_bundle scores
```

## Evaluation signals

- Labelled subnetwork files exist for each of the four cell transitions (hESC-EB, EB-Rosettes, Rosettes-hNPC, hNPC-Neurons)
- Each subnetwork file contains GLASSO edges annotated with PCA scores for the corresponding transition
- The hESC-to-EB subnetwork qualitatively contains glutamate, GABA, and related metabolites, reproducing the reported transition-specific metabolome structure
- The full-dataset GLASSO run used alpha = 0.56 with 250 iterations and per-transition runs used alpha = 0.9, matching the published settings
- PCA on the normalized dataset produced 10 components with the first three retained, as specified in the workflow

## Limitations

- The article states no explicit MetaboLINK version or complete parameter documentation in the discussion, and no software versions for the PCA/GLASSO implementations are given, hindering exact reproducibility
- The pathway database(s) used for validation are not fully specified in the discussion text, so subnetwork validation against external resources must be reconstructed from the methods (KEGG, HMDB, BMRB)
- The method was validated only on NMR-derived metabolomics of hESC-derived neural cell populations; applicability to other data types is asserted as potential (cancer, pharmacometabolomics) but not demonstrated

## Evidence

- [intro] Our method seamlessly combines principal component analysis (PCA) loadings with the graphical lasso (GLASSO) to reveal subnetworks in metabolomics datasets: "Our method seamlessly combines principal component analysis (PCA) loadings with the graphical lasso (GLASSO) to reveal subnetworks in metabolomics datasets"
- [methods] We ran PCA over the mean- and stddev-normalized dataset to produce 10 components, of which we use the first three.: "We ran PCA over the mean- and stddev-normalized dataset to produce 10 components, of which we use the first three."
- [methods] GLASSO is run with an alpha selected by cross-validation via the sklearn.covariance.GraphLassoCV implementation: "GLASSO is run with an alpha selected by cross-validation via the sklearn.covariance.GraphLassoCV implementation"
- [methods] We use the PCA and GLASSO implementations from scikit-learn,1 found in the sklearn.covariance packages, respectively.: "We use the PCA and GLASSO implementations from scikit-learn,1 found in the sklearn.covariance packages, respectively."
- [readme] A python package to visualize networks. Graphs generated by GLASSO are overlaid with PCA scores to identify significant sub-networks.: "A python package to visualize networks. Graphs generated by GLASSO are overlaid with PCA scores to identify significant sub-networks."
- [intro] We validated the effectiveness of PCA-GLASSO (referred to as MetaboLINK from now on) for metabolomics analysis of human embryonic stem cell (hESC)-derived neural cell populations.: "We validated the effectiveness of PCA-GLASSO (referred to as MetaboLINK from now on) for metabolomics analysis of human embryonic stem cell (hESC)-derived neural cell populations."
