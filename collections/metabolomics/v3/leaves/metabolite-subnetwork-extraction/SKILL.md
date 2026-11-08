---
name: metabolite-subnetwork-extraction
description: Use when analyzing high-dimensional longitudinal metabolomics concentration
  tables (e.g., 1H-NMR profiles of sequential cell populations) and you need to identify
  which metabolites and covariance relationships define a specific biological transition.
when_to_use_negative:
- Input is raw NMR spectra or peak lists rather than a normalized metabolite concentration
  table — spectra must first be pre-processed with TopSpin/Chenomx for peak identification
- The analysis goal is pattern-based metabolite identification over a predefined concentration-change
  profile, for which the MetaboAnalyst Pattern Hunter filter step is the appropriate
  choice
- The dataset is not longitudinal across ordered cell states or transitions, since
  the method derives subnetworks from sequential cell-type pairs
edam_operation: http://edamontology.org/operation_3925
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0091
tools:
- name: MetaboLINK (PCA-GLASSO algorithm)
  role: Core algorithm combining PCA loadings with GLASSO to reveal and extract transition-specific
    subnetworks
  repo: github.com/jlichtarge/pcaGLASSO
- name: pcaglasso
  role: 'Python package that visualizes networks: GLASSO-generated graphs overlaid
    with PCA scores to identify significant sub-networks'
  repo: github.com/jlichtarge/pcaGLASSO
- name: scikit-learn
  role: Provides the PCA implementation and the sklearn.covariance GraphLassoCV GLASSO
    implementation used for network estimation
- name: matplotlib
  role: Used for all plotting, including visualizing the GLASSO network overlaid with
    PCA scores
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.3389/fnins.2024.1520982
    title: MetaboLINK is a novel algorithm for unveiling cell-specific metabolic pathways
      in longitudinal datasets
  source_content_hash: sha256:c1b832dae05e9763be737ab6866db5a40911e21891aa2b46b3b7a09c25c83a11
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MetaboLINK (PCA-GLASSO algorithm)
  - Python
  - matplotlib
  - pcaglasso
  - scikit-learn
  edam_operation: http://edamontology.org/operation_3925
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# metabolite-subnetwork-extraction

## Summary

MetaboLINK workflow that overlays per-metabolite PCA loading scores onto a GLASSO-estimated sparse covariance network, then extracts transition-specific metabolite subnetworks (e.g., the hESC-to-EB glutamate/GABA subnetwork) from longitudinal NMR metabolomics data.

## When to use

Use when analyzing high-dimensional longitudinal metabolomics concentration tables (e.g., 1H-NMR profiles of sequential cell populations) and you need to identify which metabolites and covariance relationships define a specific biological transition.

## When NOT to use

- Input is raw NMR spectra or peak lists rather than a normalized metabolite concentration table — spectra must first be pre-processed with TopSpin/Chenomx for peak identification
- The analysis goal is pattern-based metabolite identification over a predefined concentration-change profile, for which the MetaboAnalyst Pattern Hunter filter step is the appropriate choice
- The dataset is not longitudinal across ordered cell states or transitions, since the method derives subnetworks from sequential cell-type pairs

## Inputs

- Mean- and stddev-normalized metabolite concentration table (n = 90 metabolites) across the five neural cell populations (hESC, EB, Rosettes, hNPC, Neurons), derived from 1H-NMR spectra acquired with a Bruker Avance 800 MHz spectrometer and identified via Chenomx
- Sequential cell-transition pair definitions (hESC-EB, EB-Rosettes, Rosettes-hNPC, hNPC-Neurons)

## Outputs

- GLASSO sparse covariance network of metabolite nodes and edges (positive and negative correlations) for the full dataset and each transition pair
- Transition-specific labelled subnetwork files containing nodes, edges, and per-node PCA-score annotations
- Matplotlib visualizations of the PCA-score-overlaid GLASSO network

## How to apply

Load the metabolite concentration table (n = 90 metabolites across the five neural cell populations) with pandas and apply mean- and stddev-normalization. Run PCA via the 'pca_bundle' class wrapping scikit-learn, producing 10 components of which the first three are used; additionally run PCA over each sequential cell-type pair (hESC-EB, EB-Rosettes, Rosettes-hNPC, hNPC-Neurons) to get a binary classification associating metabolites with one of the two cell types. Run GLASSO on the full dataset via sklearn.covariance.GraphLassoCV with alpha = 0.56 and 250 iterations, and on each truncated cell-transition pair with alpha = 0.9, yielding a sparse covariance network capturing positive and negative correlations. Overlay the per-feature PCA scores onto the GLASSO network edges with matplotlib, then extract transition-specific subnetworks (such as the reported hESC-to-EB subnetwork containing glutamate, GABA, and related metabolites) and write each as a labelled subnetwork file with nodes, edges, and PCA-score annotations. Validate that the labelled subnetwork files exist, contain GLASSO edges annotated with PCA scores for all four cell transitions, and qualitatively reproduce the reported transition-specific metabolome structure.

## Related tools

- **MetaboLINK (PCA-GLASSO algorithm)** (Core algorithm combining PCA loadings with GLASSO to reveal and extract transition-specific subnetworks) — github.com/jlichtarge/pcaGLASSO
- **pcaglasso** (Python package that visualizes networks: GLASSO-generated graphs overlaid with PCA scores to identify significant sub-networks) — github.com/jlichtarge/pcaGLASSO
- **scikit-learn** (Provides the PCA implementation and the sklearn.covariance GraphLassoCV GLASSO implementation used for network estimation)
- **matplotlib** (Used for all plotting, including visualizing the GLASSO network overlaid with PCA scores)

## Examples

```
from sklearn.covariance import GraphLassoCV; gl = GraphLassoCV(); gl.fit(normalized_metabolite_table); net = gl.covariance_
```

## Evaluation signals

- Labelled subnetwork files exist for each of the four cell transitions (hESC-EB, EB-Rosettes, Rosettes-hNPC, hNPC-Neurons)
- Each subnetwork file contains GLASSO edges annotated with PCA scores per metabolite
- The extracted hESC-to-EB subnetwork contains glutamate, GABA, and related metabolites, qualitatively reproducing the reported transition-specific metabolome structure
- Full-dataset GLASSO run used alpha = 0.56 with 250 iterations; truncated transition-pair GLASSO runs used alpha = 0.9
- PCA produced 10 components of which the first three were retained, per the article's stated parameters

## Limitations

- The MetaboLINK repository URL and software/parameter versions are not stated in the discussion text, hindering reproducibility ('The MetaboLINK code is publicly available.4' with footnote missing)
- No accession or availability statement is given for the analyzed metabolomics dataset (hESC, hNPC, neuron samples)
- The pathway database(s) used for validation of the uncovered networks are not named in the discussion
- GLASSO alpha values are transition-dependent (0.56 full dataset vs 0.9 per pair), so parameters must be re-selected for other datasets rather than copied blindly

## Evidence

- [methods] PCA over mean- and stddev-normalized dataset: "We ran PCA over the mean- and stddev-normalized dataset to produce 10 components, of which we use the first three."
- [methods] GLASSO via GraphLassoCV: "GLASSO is run with an alpha selected by cross-validation via the sklearn.covariance.GraphLassoCV implementation"
- [readme] pcaglasso package purpose: "A python package to visualize networks. Graphs generated by GLASSO are overlaid with PCA scores to identify significant sub-networks."
- [intro] Combining PCA loadings with GLASSO: "Our method seamlessly combines principal component analysis (PCA) loadings with the graphical lasso (GLASSO) to reveal subnetworks in metabolomics datasets"
- [abstract] Dimensionality reduction rationale: "This method is designed to reduce the dimensionality of large datasets while preserving significant variance."
- [methods] pca_bundle class: "The class ‘pca_bundle’ runs PCA and allows the easy visualization of PCA-related graphs, showing loadings, scores, and explained variance."
