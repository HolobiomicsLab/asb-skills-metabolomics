---
name: co-culture-ground-truth-annotation
description: Use when working with SpaceM co-culture MORPHnMOL.csv matrices containing
  microscopy fluorescence channels (mCherry/GFP) and needing per-cell species/cell-type
  labels for supervised classifiers like LDA.
when_to_use_negative:
- Input is a single-cell-type culture (e.g., the isogenic dHepaRG NAFLD model) with
  no fluorescence-based species marker — use metabolic-state clustering instead of
  fluorescence-based ground-truth annotation
- The spatio-molecular matrix lacks fluorescence channels (no co-registered microscopy),
  leaving no orthogonal basis for label assignment
- Goal is de novo cell-type discovery from metabolomes alone — supervised labels would
  bias the analysis
edam_operation: http://edamontology.org/operation_3791
edam_topics:
- http://edamontology.org/topic_2229
- http://edamontology.org/topic_3520
tools:
- name: SpaceM
  role: Produces the MORPHnMOL.csv spatio-molecular matrix whose fluorescence columns
    and metabolite intensities the labeling operates on
  repo: github.com/alexandrovteam/SpaceM
- name: Python
  role: Implements threshold-based label assignment in pandas and downstream scikit-learn
    LDA classification
- name: SCANPY
  role: Downstream analysis of the labeled single-cell matrix
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41592-021-01198-0
    title: SpaceM reveals metabolic states of single cells
  source_content_hash: sha256:5ae3ead88de7e3a517a95df2096607c3295859dd4ad9938cbf433a4b3a789d2d
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Python
  - SCANPY
  - SpaceM
  edam_operation: http://edamontology.org/operation_3791
  edam_topics:
  - http://edamontology.org/topic_2229
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# co-culture-ground-truth-annotation

## Summary

Assign ground-truth cell-type labels to co-cultured HeLa and NIH3T3 cells in SpaceM spatio-molecular matrices using fluorescence thresholds (mCherry for HeLa, GFP for NIH3T3), enabling supervised classification of single-cell metabolic profiles.

## When to use

Use when working with SpaceM co-culture MORPHnMOL.csv matrices containing microscopy fluorescence channels (mCherry/GFP) and needing per-cell species/cell-type labels for supervised classifiers like LDA.

## When NOT to use

- Input is a single-cell-type culture (e.g., the isogenic dHepaRG NAFLD model) with no fluorescence-based species marker — use metabolic-state clustering instead of fluorescence-based ground-truth annotation
- The spatio-molecular matrix lacks fluorescence channels (no co-registered microscopy), leaving no orthogonal basis for label assignment
- Goal is de novo cell-type discovery from metabolomes alone — supervised labels would bias the analysis

## Inputs

- SpaceM spatio-molecular matrix MORPHnMOL.csv (per-cell normalized metabolite intensities + microscopy fluorescence channels)
- mCherry and GFP fluorescence intensity columns from co-registered microscopy
- Threshold/decision boundary separating mCherry-positive (HeLa) from GFP-positive (NIH3T3) cells

## Outputs

- Per-cell ground-truth cell-type label column (HeLa vs NIH3T3)
- Feature-restricted metabolite intensity table (88 published features) paired with labels
- CSV of per-cell metabolite intensities, cell-type labels, and cross-validation accuracy for the downstream classifier

## How to apply

Load the MORPHnMOL.csv spatio-molecular matrix, which SpaceM generates per cell with normalized metabolic intensities plus microscopy-derived phenotypic properties such as cell fluorescence intensities. Locate the mCherry and GFP fluorescence intensity columns and apply a linear decision boundary in the mCherry-versus-GFP plane: cells above the mCherry threshold are labeled HeLa (human) and cells above the GFP threshold are labeled NIH3T3 (mouse fibroblast). The fluorescence channels are trustworthy labels because they are orthogonal to the metabolite features — the 88 metabolite intensity columns used for prediction carry no species-specific fluorescence information. After labeling, restrict features to the published metabolite columns before fitting the supervised classifier (e.g., scikit-learn LinearDiscriminantAnalysis with RepeatedStratifiedKFold cross-validation). Validate the annotation pipeline by confirming the downstream classifier reproduces the reported ~91.3% cell-type prediction accuracy from single-cell metabolomes alone.

## Related tools

- **SpaceM** (Produces the MORPHnMOL.csv spatio-molecular matrix whose fluorescence columns and metabolite intensities the labeling operates on) — github.com/alexandrovteam/SpaceM
- **Python** (Implements threshold-based label assignment in pandas and downstream scikit-learn LDA classification)
- **SCANPY** (Downstream analysis of the labeled single-cell matrix)

## Examples

```
df = pd.read_csv('MORPHnMOL.csv'); df['label'] = np.where(df['mCherry'] > df['GFP'], 'HeLa', 'NIH3T3')
```

## Evaluation signals

- Downstream 10-fold repeated stratified cross-validation of LDA on the 88 metabolite features reproduces ~91.3% accuracy for HeLa vs NIH3T3 prediction
- Every cell in MORPHnMOL.csv receives exactly one non-null label (mCherry-positive or GFP-positive)
- Label distribution matches the expected co-culture composition seeded in the experiment
- Labels are perfectly concordant with the fluorescence decision boundary when re-plotted in the mCherry-versus-GFP plane
- Prediction uses only metabolite columns — no fluorescence/morphology features leak into the classifier feature set

## Limitations

- Co-sampling of neighboring cells by the same ablation mark minimally affects but does not eliminate error in cell-type prediction in spatially-heterogeneous co-cultures
- Requires spatially separated, fluorescence-distinguishable cell populations; cannot annotate cells lacking distinct fluorescence markers
- The linear fluorescence decision boundary assumes bimodal, well-separated mCherry/GFP intensity distributions; intermediate/double-positive cells are ambiguous

## Evidence

- [intro] SpaceM generates a spatio-molecular matrix for each cell containing a normalized metabolic profile and microscopy-derived phenotypic properties such as cell fluorescence intensities: "SpaceM generates a spatio-molecular matrix for each cell containing a normalized metabolic profile and microscopy-derived phenotypic properties such as cell fluorescence intensities"
- [intro] the cell types were predicted with an accuracy of 91.3% solely based on the single-cell metabolomes: "the cell types were predicted with an accuracy of 91.3% solely based on the single-cell metabolomes"
- [discussion] the co-sampling only minimally affects the accuracy of the cell type prediction in spatially-heterogeneous co-cultures: "the co-sampling only minimally affects the accuracy of the cell type prediction in spatially-heterogeneous co-cultures"
