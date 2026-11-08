---
name: metabolite-network-edge-list-generation
description: Use when longitudinal metabolomics data (e.g., NMR-derived concentration
  matrices over sequential cell-state transitions) must be converted into sparse metabolite
  networks with per-transition edge lists.
when_to_use_negative:
- The input is raw 1D/2D NMR spectra requiring peak identification and metabolite
  quantification (use TopSpin/Chenomx preprocessing first)
- The goal is single-timepoint differential abundance testing rather than pairwise-transition
  network structure
- Pairwise datasets are large enough that GraphLassoCV cross-validation is stable
  — then select alpha by cross-validation instead of fixing alpha = 0.9
edam_topics:
- http://edamontology.org/topic_3382
tools:
- name: MetaboLINK (PCA-GLASSO algorithm)
  role: The overarching pipeline whose GLASSO stage produces the sparse precision
    matrix from which edge lists are extracted
  repo: github.com/jlichtarge/pcaGLASSO
- name: pcaglasso
  role: Python package used to visualize the GLASSO-generated graphs overlaid with
    PCA scores to identify significant subnetworks
  repo: github.com/jlichtarge/pcaGLASSO
- name: scikit-learn
  role: Provides the sklearn.covariance GraphLasso/GraphLassoCV implementations used
    to fit the sparse covariance networks
- name: matplotlib
  role: Used for all plotting of the per-transition metabolite networks
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.3389/fnins.2024.1520982
    title: MetaboLINK is a novel algorithm for unveiling cell-specific metabolic pathways
      in longitudinal datasets
  source_content_hash: sha256:55526b9354c1acd69c64f70565eae97541f3f391037d757c8e726d210e6e0fc8
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MetaboLINK (PCA-GLASSO algorithm)
  - Python
  - matplotlib
  - pcaglasso
  - scikit-learn
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3382
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9473684210526315
n_claims: 19
---

# metabolite-network-edge-list-generation

## Summary

Generates sparse metabolite co-dependency networks by fitting GLASSO precision matrices to metabolite concentration matrices (e.g., 90 NMR-derived metabolites across neural differentiation stages) and extracting non-zero off-diagonal partial correlations into per-transition edge lists.

## When to use

Use when longitudinal metabolomics data (e.g., NMR-derived concentration matrices over sequential cell-state transitions) must be converted into sparse metabolite networks with per-transition edge lists.

## When NOT to use

- The input is raw 1D/2D NMR spectra requiring peak identification and metabolite quantification (use TopSpin/Chenomx preprocessing first)
- The goal is single-timepoint differential abundance testing rather than pairwise-transition network structure
- Pairwise datasets are large enough that GraphLassoCV cross-validation is stable — then select alpha by cross-validation instead of fixing alpha = 0.9

## Inputs

- NMR-derived metabolite concentration matrix (n = 90 metabolites across five neural cell populations, Supplementary Table S1)
- Sequential cell-transition pair definitions (hESC-to-EB, EB-to-Rosettes, Rosettes-to-hNPC, hNPC-to-Neuron)
- Fixed L1 regularization parameter (alpha = 0.9)

## Outputs

- One metabolite edge list per transition pair (non-zero off-diagonal GLASSO entries, both positive and negative partial correlations)
- Cross-transition comparison summary table identifying transition-specific subnetworks
- Per-transition GLASSO network visualizations (matplotlib), optionally with PCA scores overlaid on nodes

## How to apply

Load the NMR-derived metabolite concentration matrix (Supplementary Table S1: n = 90 metabolites across five neural cell populations) with pandas and truncate it to each transition pair (hESC-to-EB, EB-to-Rosettes, Rosettes-to-hNPC, hNPC-to-Neuron). For each pair, fit a sparse covariance network with sklearn.covariance's GraphLasso, fixing the L1 regularization parameter alpha = 0.9 because cross-validation via GraphLassoCV is unstable on the small pairwise datasets. Extract only the non-zero off-diagonal entries of the GLASSO precision/covariance matrix, retaining both positive and negative partial correlations and dropping features with no edges, yielding one edge list per transition. Compare the four edge lists to isolate transition-specific subnetworks, mapping hallmark metabolites to stages (glutamate/GABA for hESCs, glycerol/serine for rosettes, branched-chain amino acids for hNPCs, lipids/glutathione for neurons), and visualize with matplotlib, optionally overlaying pairwise PCA scores on network nodes.

## Related tools

- **MetaboLINK (PCA-GLASSO algorithm)** (The overarching pipeline whose GLASSO stage produces the sparse precision matrix from which edge lists are extracted) — github.com/jlichtarge/pcaGLASSO
- **pcaglasso** (Python package used to visualize the GLASSO-generated graphs overlaid with PCA scores to identify significant subnetworks) — github.com/jlichtarge/pcaGLASSO
- **scikit-learn** (Provides the sklearn.covariance GraphLasso/GraphLassoCV implementations used to fit the sparse covariance networks)
- **matplotlib** (Used for all plotting of the per-transition metabolite networks)

## Examples

```
from sklearn.covariance import GraphLasso; model = GraphLasso(alpha=0.9).fit(X_transition_pair); edges = [(i,j) for i in range(p) for j in range(i+1,p) if model.precision_[i,j] != 0]
```

## Evaluation signals

- Each transition's edge list contains only non-zero off-diagonal precision/covariance entries; features with no edges are dropped
- Both positive and negative partial correlations are retained in each edge list (no sign filtering)
- Expert review of the summary table confirms hallmark metabolites appear in the correct stage's subnetwork: glutamate/GABA edges in the hESC transition, lipid/glutathione edges in the neuron transition, glycerol/serine in rosettes, branched-chain amino acids in hNPCs
- The hESC-to-EB subnetwork includes glutamate, 4-aminobutyrate (GABA), and 2-phosphoglycerate
- Exactly four edge lists are produced, one per sequential transition pair, written to tabular output files

## Limitations

- Cross-validation for alpha (GraphLassoCV) is unstable on the small pairwise datasets, forcing a fixed alpha = 0.9 rather than data-driven regularization selection
- PCA/GLASSO implementations are taken from scikit-learn, but no software versions or parameter settings beyond alpha are stated in the article
- Edge lists are derived from a single fixed regularization choice; edge presence may be sensitive to alpha

## Evidence

- [methods] GLASSO edge extraction with fixed alpha: "We use the PCA and GLASSO implementations from scikit-learn,1 found in the sklearn.covariance packages, respectively."
- [methods] Regularization selection rationale: "GLASSO is run with an alpha selected by cross-validation via the sklearn.covariance.GraphLassoCV implementation"
- [readme] Network visualization with PCA overlay: "Graphs generated by GLASSO are overlaid with PCA scores to identify significant sub-networks."
- [methods] Plotting toolchain: "All plotting is done using matplotlib,2"
- [intro] Subnetwork discovery goal: "Our method seamlessly combines principal component analysis (PCA) loadings with the graphical lasso (GLASSO) to reveal subnetworks in metabolomics datasets"
