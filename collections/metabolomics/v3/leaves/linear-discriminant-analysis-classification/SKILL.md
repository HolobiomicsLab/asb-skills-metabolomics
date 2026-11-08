---
name: linear-discriminant-analysis-classification
description: Use when a SpaceM spatio-molecular matrix (MORPHnMOL.csv) contains single-cell
  metabolite intensities plus fluorescence channels for two or more morphologically
  distinguishable cell types, and the goal is to test whether metabolic profiles alone
  predict cell type.
when_to_use_negative:
- Input is raw MALDI-imaging data (.RAW/.imzML/.ibd microscopy tiles) that has not
  yet been processed by SpaceM into a per-cell spatio-molecular matrix — run the SpaceM
  pipeline first.
- Cells cannot be assigned ground-truth labels by an independent modality (e.g. no
  fluorescence channels or marker decision boundary), making supervised classification
  unverifiable.
- Classes are defined by the same features used for prediction, which would inflate
  accuracy and not validate single-cell metabolic resolution.
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_2229
tools:
- name: SpaceM
  role: Produces the spatio-molecular matrix (MORPHnMOL.csv) of normalized per-cell
    metabolic profiles and fluorescence properties that feeds the LDA classifier
  repo: https://github.com/alexandrovteam/SpaceM
- name: SCANPY
  role: Single-cell analysis toolbox used for downstream processing of the spatio-molecular
    matrices
- name: Python
  role: Runs the classification workflow (pandas loading, scikit-learn LDA, RepeatedStratifiedKFold
    cross-validation)
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41592-021-01198-0
    title: SpaceM reveals metabolic states of single cells
  source_content_hash: sha256:f84f19388254db31682d1d66225949f18cf4b3fcdf9ea493b3a1a945871b7877
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/alexandrovteam/SpaceM
  tools:
  - Python
  - SCANPY
  - SpaceM
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_2229
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# linear-discriminant-analysis-classification

## Summary

Classify cell types in co-culture solely from SpaceM single-cell metabolic profiles using Linear Discriminant Analysis on metabolite intensity columns of the spatio-molecular matrix, validating single-cell resolution of the method. In the SpaceM HeLa/NIH3T3 benchmark this reached 91.3% accuracy.

## When to use

Use when a SpaceM spatio-molecular matrix (MORPHnMOL.csv) contains single-cell metabolite intensities plus fluorescence channels for two or more morphologically distinguishable cell types, and the goal is to test whether metabolic profiles alone predict cell type.

## When NOT to use

- Input is raw MALDI-imaging data (.RAW/.imzML/.ibd microscopy tiles) that has not yet been processed by SpaceM into a per-cell spatio-molecular matrix — run the SpaceM pipeline first.
- Cells cannot be assigned ground-truth labels by an independent modality (e.g. no fluorescence channels or marker decision boundary), making supervised classification unverifiable.
- Classes are defined by the same features used for prediction, which would inflate accuracy and not validate single-cell metabolic resolution.

## Inputs

- SpaceM spatio-molecular matrix MORPHnMOL.csv (per-cell normalized metabolite intensities + microscopy-derived fluorescence intensities and morphological features)
- Ground-truth cell-type labels derived from mCherry/GFP fluorescence decision boundary

## Outputs

- CSV file with per-cell metabolite intensities, cell-type labels, and the reported cross-validation accuracy
- Fitted LinearDiscriminantAnalysis classifier on the 88 metabolite features
- Estimated classification accuracy (~91.3% expected for the HeLa/NIH3T3 co-culture)

## How to apply

Load MORPHnMOL.csv (the final SpaceM spatio-molecular matrix) into pandas, retaining metabolite intensity columns and microscopy-derived fluorescence channels. Assign ground-truth labels using a linear decision boundary in the mCherry-versus-GFP fluorescence intensities (HeLa = mCherry-positive, NIH3T3 = GFP-positive), so that labels come from an independent modality rather than the metabolites themselves. Select the published feature set of 88 metabolite intensities, fit scikit-learn's LinearDiscriminantAnalysis on those features, and evaluate with RepeatedStratifiedKFold 10-fold cross-validation. LDA is appropriate because it finds linear discriminants between cell-type classes directly in metabolite intensity space, mirroring the linear discriminant (LD) axes SpaceM uses downstream for state analysis. Write a CSV with per-cell intensities, labels, and the cross-validation accuracy.

## Related tools

- **SpaceM** (Produces the spatio-molecular matrix (MORPHnMOL.csv) of normalized per-cell metabolic profiles and fluorescence properties that feeds the LDA classifier) — https://github.com/alexandrovteam/SpaceM
- **SCANPY** (Single-cell analysis toolbox used for downstream processing of the spatio-molecular matrices)
- **Python** (Runs the classification workflow (pandas loading, scikit-learn LDA, RepeatedStratifiedKFold cross-validation))

## Examples

```
from sklearn.discriminant_analysis import LinearDiscriminantAnalysis; from sklearn.model_selection import RepeatedStratifiedKFold, cross_val_score; lda = LinearDiscriminantAnalysis(); scores = cross_val_score(lda, X88, y_fluorescence_labels, cv=RepeatedStratifiedKFold(n_splits=10, n_repeats=5))
```

## Evaluation signals

- Repeated stratified 10-fold cross-validation accuracy of approximately 91.3% for HeLa vs NIH3T3 prediction from the 88 metabolite intensities
- Output CSV contains per-cell metabolite intensities, cell-type labels, and the cross-validation accuracy result
- Ground-truth labels agree with the linear mCherry-vs-GFP fluorescence decision boundary (HeLa mCherry-positive, NIH3T3 GFP-positive)
- Feature set matches the published 88 metabolite columns from the matrix

## Limitations

- Co-sampling of neighboring cells by the same ablation mark occurs in spatially-heterogeneous co-cultures, though the paper reports it only minimally affects cell-type prediction accuracy.
- Accuracy depends on segmentation quality: the pipeline requires CellProfiler project files able to segment the cells and ablation marks that are visible and non-overlapping in post-MALDI microscopy.
- SpaceM's tile-stitching code is optimized for Nikon Ti-E microscope output format, a limitation of the implementation rather than the method.

## Evidence
