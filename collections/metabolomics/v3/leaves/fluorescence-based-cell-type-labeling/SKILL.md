---
name: fluorescence-based-cell-type-labeling
description: Use when a SpaceM MORPHnMOL.csv spatio-molecular matrix contains microscopy-derived
  fluorescence channels (e.g., mCherry for HeLa, GFP for NIH3T3) and per-cell cell-type
  ground truth is needed to train or evaluate a metabolic-profile classifier.
when_to_use_negative:
- Cells lack genetically encoded fluorescent reporters or matched fluorescence microscopy
  channels, so no fluorescence-based ground truth exists
- Input already contains trusted per-cell type annotations, making fluorescence re-labeling
  redundant
- The goal is unsupervised discovery of metabolic states (e.g., clustering dHepaRG
  hepatocytes), where fluorescence labels would bias the analysis
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_3391
tools:
- name: SpaceM
  role: Produces the spatio-molecular matrix (MORPHnMOL.csv) whose fluorescence-intensity
    columns supply the labeling features
  repo: github.com/alexandrovteam/SpaceM
- name: CellProfiler
  role: Segments cells in pre/post-MALDI microscopy so per-cell fluorescence intensities
    can be measured for labeling
- name: SCANPY
  role: Downstream analysis of the labeled spatio-molecular matrix
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41592-021-01198-0
    title: SpaceM reveals metabolic states of single cells
  source_content_hash: sha256:3bc42461663145e30f205ba97df87d3c0cbebee503d137c7df142c4a5e1ca8ee
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - CellProfiler
  - Python
  - SCANPY
  - SpaceM
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_3391
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# fluorescence-based-cell-type-labeling

## Summary

Assigns ground-truth cell-type labels in co-culture single-cell metabolomics by applying a linear decision boundary on microscopy-derived fluorescence intensities (e.g., mCherry-vs-GFP channels) from the SpaceM spatio-molecular matrix, providing labels for classifier training and validation.

## When to use

Use when a SpaceM MORPHnMOL.csv spatio-molecular matrix contains microscopy-derived fluorescence channels (e.g., mCherry for HeLa, GFP for NIH3T3) and per-cell cell-type ground truth is needed to train or evaluate a metabolic-profile classifier.

## When NOT to use

- Cells lack genetically encoded fluorescent reporters or matched fluorescence microscopy channels, so no fluorescence-based ground truth exists
- Input already contains trusted per-cell type annotations, making fluorescence re-labeling redundant
- The goal is unsupervised discovery of metabolic states (e.g., clustering dHepaRG hepatocytes), where fluorescence labels would bias the analysis

## Inputs

- SpaceM spatio-molecular matrix MORPHnMOL.csv with per-cell normalized metabolic profiles and microscopy-derived fluorescence intensity columns
- Pre/post-MALDI microscopy tiles used by CellProfiler segmentation to define per-cell fluorescence reads
- Knowledge of the reporter channels per cell line (e.g., mCherry-positive HeLa, GFP-positive NIH3T3)

## Outputs

- Per-cell cell-type label column aligned to matrix row order
- CSV of per-cell metabolite intensities with assigned cell-type labels for classifier training/evaluation
- Label balance summary (counts per cell type) for sanity checking

## How to apply

Load the spatio-molecular matrix (MORPHnMOL.csv) from the scAnalysis sub-folder, which SpaceM generates with normalized metabolic profiles plus microscopy-derived phenotypic properties including cell fluorescence intensities. Identify the fluorescence channels corresponding to each cell type's reporter (mCherry for HeLa, GFP for NIH3T3 in the co-culture dataset). Apply the linear decision boundary in the mCherry-versus-GFP fluorescence intensities that separates the two cell types, assigning each cell a binary label. Because SpaceM co-registers microscopy images of cells with MALDI laser ablation marks, fluorescence labels can be matched to individually ablated cells; be aware that ablation marks co-sampling neighboring cells only minimally affects cell-type prediction accuracy in spatially heterogeneous co-cultures, per the paper's discussion. Use the resulting labels as ground truth for supervised models (e.g., LDA on the 88 selected metabolite features) and cross-check label balance across cell types before training.

## Related tools

- **SpaceM** (Produces the spatio-molecular matrix (MORPHnMOL.csv) whose fluorescence-intensity columns supply the labeling features) — github.com/alexandrovteam/SpaceM
- **CellProfiler** (Segments cells in pre/post-MALDI microscopy so per-cell fluorescence intensities can be measured for labeling)
- **SCANPY** (Downstream analysis of the labeled spatio-molecular matrix)

## Examples

```
df = pd.read_csv('MF/Analysis/scAnalysis/MORPHnMOL.csv'); df['cell_type'] = np.where((df['mCherry'] > t) & (df['GFP'] <= t), 'HeLa', 'NIH3T3')
```

## Evaluation signals

- Every cell receives exactly one label after the mCherry-vs-GFP boundary is applied (no unassigned rows in the label column)
- Labels assigned via fluorescence reproduce the reported ~91.3% LDA cross-validation accuracy when used as ground truth for the 88-metabolite classifier
- Repeated stratified 10-fold CV (RepeatedStratifiedKFold) accuracy computed on fluorescence labels is stable across folds
- Per-cell label counts reflect the expected co-culture proportions, with no collapse to a single class

## Limitations

- Ablation marks co-sampling neighboring cells can mix metabolic signal across cell types; the paper reports this only minimally affects cell-type prediction accuracy in spatially heterogeneous co-cultures
- No version numbers are given for CellProfiler, Fiji, or Python in the README, so reproducing the microscopy/segmentation stack exactly may be ambiguous
- Fluorescence ground truth depends on visible, non-overlapping ablation marks in post-MALDI microscopy and stitched tile coordinates (out.txt), optimized for Nikon Ti-E output

## Evidence
