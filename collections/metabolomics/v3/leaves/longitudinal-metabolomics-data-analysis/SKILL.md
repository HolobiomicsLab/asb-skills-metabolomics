---
name: longitudinal-metabolomics-data-analysis
description: Use when analyzing metabolite abundance tables collected across developmental
  stages or time points (e.g., 1H-NMR metabolite concentrations from sequential cell
  populations) to reduce dimensionality and identify cell-specific metabolic subnetworks.
when_to_use_negative:
- Data are not longitudinal/staged (e.g., a single condition with no time or developmental
  axis), so PCA loadings cannot be linked to stage-specific transitions
- The goal is targeted pathway enrichment on a curated metabolite list rather than
  unsupervised dimensionality reduction and network inference on the full metabolite
  table
- Input is raw NMR spectra (FIDs/free induction decays) rather than a quantified metabolite
  concentration table — spectral pre-processing and peak identification (TopSpin/Chenomx)
  must be completed first
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3272
- http://edamontology.org/topic_0154
tools:
- name: MetaboLINK (PCA-GLASSO algorithm)
  role: 'Core algorithm: combines PCA loadings with GLASSO to reduce dimensionality
    and reveal sub-networks in longitudinal metabolomics data'
  repo: github.com/jlichtarge/pcaGLASSO
- name: pcaglasso
  role: Python package implementing the workflow; 'pca_bundle' class runs PCA and
    visualizes loadings, scores, and explained variance
  repo: github.com/jlichtarge/pcaGLASSO
- name: scikit-learn
  role: Provides the PCA and GLASSO implementations (sklearn.covariance, GraphLassoCV
    with cross-validated alpha)
- name: Chenomx
  role: Peak identification and metabolite quantification from 1H-NMR spectra using
    the 800 MHz library (380 metabolites)
- name: matplotlib
  role: All plotting of PCA scores plots and GLASSO network overlays
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.3389/fnins.2024.1520982
    title: MetaboLINK is a novel algorithm for unveiling cell-specific metabolic pathways
      in longitudinal datasets
  source_content_hash: sha256:3c228ff6bc55e1bb903ca9672196040f0a4d3e3d3bb66acdc25c0466bc135f6e
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Chenomx
  - MetaboLINK (PCA-GLASSO algorithm)
  - Python
  - matplotlib
  - pandas
  - pcaglasso
  - scikit-learn
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3272
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# longitudinal-metabolomics-data-analysis

## Summary

Analysis of longitudinal NMR-based metabolomics data using the MetaboLINK PCA-GLASSO approach, which combines PCA dimensionality reduction with Graphical Lasso network inference to reveal cell-stage-specific metabolic pathways and subnetworks across developmental time points.

## When to use

Use when analyzing metabolite abundance tables collected across developmental stages or time points (e.g., 1H-NMR metabolite concentrations from sequential cell populations) to reduce dimensionality and identify cell-specific metabolic subnetworks.

## When NOT to use

- Data are not longitudinal/staged (e.g., a single condition with no time or developmental axis), so PCA loadings cannot be linked to stage-specific transitions
- The goal is targeted pathway enrichment on a curated metabolite list rather than unsupervised dimensionality reduction and network inference on the full metabolite table
- Input is raw NMR spectra (FIDs/free induction decays) rather than a quantified metabolite concentration table — spectral pre-processing and peak identification (TopSpin/Chenomx) must be completed first

## Inputs

- Metabolite concentration table (e.g., n = 90 metabolites from Supplementary Table S1, quantified by Chenomx from 1H-NMR spectra, 3–8 biological replicates per cell population)
- 1H-NMR 1D spectra (30 min at 25°C on 1,000,000 cells per population) pre-processed in TopSpin and peak-identified in Chenomx
- 2D 1H-1H TOCSY NMR spectra (24 h) for J-coupled peak identification
- Longitudinal stage labels (hESCs → EBs → Rosettes → hNPCs → Neurons)

## Outputs

- PCA sample scores and loadings plus explained variance for the first three retained components (exported e.g. as pca_scores.csv and pca_score_plot.png)
- GLASSO precision-matrix graphs overlaid with PCA scores, identifying significant metabolite sub-networks
- Cell-type biomarkers and stage-specific metabolic pathways (amino acid, lipid, and energy metabolism)

## How to apply

Start from a metabolite concentration table (e.g., 90 metabolites quantified by Chenomx from 1H-NMR spectra acquired on a Bruker Avance 800 MHz spectrometer with cryoprobe, with 3–8 biological replicates per population). Normalize each metabolite feature by its mean and standard deviation, then run PCA over the normalized dataset to produce 10 components, retaining the first three. Use the 'pca_bundle' class to capture PCA loadings, sample scores, and explained variance for visualization. Run GLASSO with the regularization alpha selected by cross-validation via sklearn.covariance.GraphLassoCV, and overlay the GLASSO-generated graphs with PCA scores to identify significant sub-networks, cross-matching metabolites against KEGG, HMDB, and BMRB resources.

## Related tools

- **MetaboLINK (PCA-GLASSO algorithm)** (Core algorithm: combines PCA loadings with GLASSO to reduce dimensionality and reveal sub-networks in longitudinal metabolomics data) — github.com/jlichtarge/pcaGLASSO
- **pcaglasso** (Python package implementing the workflow; 'pca_bundle' class runs PCA and visualizes loadings, scores, and explained variance) — github.com/jlichtarge/pcaGLASSO
- **scikit-learn** (Provides the PCA and GLASSO implementations (sklearn.covariance, GraphLassoCV with cross-validated alpha))
- **Chenomx** (Peak identification and metabolite quantification from 1H-NMR spectra using the 800 MHz library (380 metabolites))
- **matplotlib** (All plotting of PCA scores plots and GLASSO network overlays)

## Examples

```
from pcaglasso import pca_bundle; pb = pca_bundle(X_zscored, n_components=10); pb.plot_scores(groups=stage_labels, use_components=[0,1,2])
```

## Evaluation signals

- The PCA scores plot clearly separates the five cell-type sample groups (hESC, EB, rosettes, hNPC, neurons) in an unsupervised manner, with each lineage clustering independently
- PCA is fit on the mean- and stddev-normalized (z-score) dataset and produces 10 components, with the first three retained
- GLASSO alpha is selected by cross-validation (GraphLassoCV) rather than set arbitrarily
- Identified metabolites are cross-matched against KEGG, HMDB, and BMRB, and validated pathways match known biology (e.g., amino acid, lipid, and energy metabolism shifts across stages)
- The pca_bundle output contains all three artifacts: loadings, scores, and explained variance per retained component

## Limitations

- No software versions or explicit parameter settings (beyond cross-validated alpha and 10 PCA components) are stated, hindering exact reproduction
- No accession or availability statement is given for the analyzed metabolomics dataset (hESC, hNPC, neuron samples), limiting input reuse
- The pathway databases used for validation are not fully named, so the validation step is not fully reproducible from the article text

## Evidence

- [intro] Our method seamlessly combines principal component analysis (PCA) loadings with the graphical lasso (GLASSO) to reveal subnetworks in metabolomics datasets: "Our method seamlessly combines principal component analysis (PCA) loadings with the graphical lasso (GLASSO) to reveal subnetworks in metabolomics datasets"
- [methods] PCA over mean/stddev-normalized data: "We ran PCA over the mean- and stddev-normalized dataset to produce 10 components, of which we use the first three."
- [methods] GLASSO cross-validated alpha: "GLASSO is run with an alpha selected by cross-validation via the sklearn.covariance.GraphLassoCV implementation"
- [methods] pca_bundle class: "The class ‘pca_bundle’ runs PCA and allows the easy visualization of PCA-related graphs, showing loadings, scores, and explained variance."
- [readme] README package description: "A python package to visualize networks. Graphs generated by GLASSO are overlaid with PCA scores to identify significant sub-networks."
- [intro] MetaboLINK validation on hESC neural populations: "We validated the effectiveness of PCA-GLASSO (referred to as MetaboLINK from now on) for metabolomics analysis of human embryonic stem cell (hESC)-derived neural cell populations."
