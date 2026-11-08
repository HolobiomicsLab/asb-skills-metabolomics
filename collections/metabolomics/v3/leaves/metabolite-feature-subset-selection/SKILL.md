---
name: metabolite-feature-subset-selection
description: Use when a SpaceM MORPHnMOL.csv spatio-molecular matrix must be reduced
  to a published or pre-defined metabolite feature subset before training a cell-type
  classifier (e.g., LDA on co-cultured HeLa/NIH3T3 cells).
when_to_use_negative:
- Input is raw MALDI-imaging data (.RAW, .imzML, .ibd) or pre/post-MALDI microscopy
  tiles — feature selection applies only after SpaceM has produced the MORPHnMOL.csv
  matrix.
- The analysis goal is unsupervised discovery of metabolic states (e.g., LD-based
  state clustering in dHepaRG hepatocytes) rather than supervised prediction with
  a defined feature set.
- Fluorescence or morphological features are intended as classifier inputs — this
  skill restricts features to metabolite intensities only.
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0091
tools:
- name: SpaceM
  role: Produces the MORPHnMOL.csv spatio-molecular matrix from which metabolite intensity
    columns are selected
  repo: github.com/alexandrovteam/SpaceM
- name: Python
  role: Loads MORPHnMOL.csv in pandas and subsets the 88 metabolite feature columns
    prior to classifier fitting
- name: SCANPY
  role: Downstream analysis toolbox for the spatio-molecular matrices after feature
    subsetting
  repo: github.com/alexandrovteam/SpaceM
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41592-021-01198-0
    title: SpaceM reveals metabolic states of single cells
  source_content_hash: sha256:c8f185f4514f9768a22be4d0c19c2c2115f343d2b42e1795333549fc6414ba49
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Python
  - SCANPY
  - SpaceM
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# metabolite-feature-subset-selection

## Summary

Selects a defined subset of metabolite intensity columns from a SpaceM spatio-molecular matrix (e.g., the 88 published metabolite features) to serve as the feature space for downstream single-cell modeling such as LDA cell-type classification.

## When to use

Use when a SpaceM MORPHnMOL.csv spatio-molecular matrix must be reduced to a published or pre-defined metabolite feature subset before training a cell-type classifier (e.g., LDA on co-cultured HeLa/NIH3T3 cells).

## When NOT to use

- Input is raw MALDI-imaging data (.RAW, .imzML, .ibd) or pre/post-MALDI microscopy tiles — feature selection applies only after SpaceM has produced the MORPHnMOL.csv matrix.
- The analysis goal is unsupervised discovery of metabolic states (e.g., LD-based state clustering in dHepaRG hepatocytes) rather than supervised prediction with a defined feature set.
- Fluorescence or morphological features are intended as classifier inputs — this skill restricts features to metabolite intensities only.

## Inputs

- SpaceM spatio-molecular matrix MORPHnMOL.csv (per-cell normalized metabolite intensities plus microscopy-derived fluorescence intensities and morpho-spatial features)
- List or column mask identifying the target metabolite feature subset (e.g., the 88 published HeLa/NIH3T3 metabolite features)

## Outputs

- Reduced per-cell feature matrix containing only the selected metabolite intensity columns
- Per-cell table pairing metabolite intensities with fluorescence-derived ground-truth cell-type labels (HeLa vs NIH3T3)

## How to apply

Load the MORPHnMOL.csv spatio-molecular matrix produced by SpaceM (found in the scAnalysis sub-folder) into pandas, keeping both metabolite intensity columns and microscopy-derived fluorescence channels. Identify the metabolite feature columns corresponding to the published feature set — for the HeLa/NIH3T3 co-culture benchmark this is the 88 metabolite features used in the published cell-type prediction. Restrict the feature matrix to these metabolite intensities, holding back the fluorescence channels (mCherry/GFP) which are used only to derive ground-truth labels via the linear decision boundary, not as classifier features. The rationale is to predict cell type solely from single-cell metabolomes, so contaminating the feature space with fluorescence would defeat the validation; the paper reports 91.3% accuracy from metabolomes alone via repeated stratified 10-fold cross-validated LDA.

## Related tools

- **SpaceM** (Produces the MORPHnMOL.csv spatio-molecular matrix from which metabolite intensity columns are selected) — github.com/alexandrovteam/SpaceM
- **Python** (Loads MORPHnMOL.csv in pandas and subsets the 88 metabolite feature columns prior to classifier fitting)
- **SCANPY** (Downstream analysis toolbox for the spatio-molecular matrices after feature subsetting) — github.com/alexandrovteam/SpaceM

## Examples

```
df = pd.read_csv('MF/Analysis/scAnalysis/MORPHnMOL.csv'); X = df[metabolite_features_88]; y = ((df['mCherry']>thresh)&(df['GFP']<=thresh)).astype(int)
```

## Evaluation signals

- The selected feature matrix contains exactly the expected number of metabolite columns (88 for the HeLa/NIH3T3 benchmark) and excludes mCherry/GFP fluorescence columns.
- Every retained column corresponds to a metabolite intensity (not a morpho-spatial or fluorescence feature) in MORPHnMOL.csv.
- Repeated stratified 10-fold cross-validated LDA on the subset reproduces the reported ~91.3% cell-type classification accuracy.
- Ground-truth labels derived from the mCherry-vs-GFP fluorescence decision boundary are kept separate from the feature matrix used for prediction.

## Limitations

- Co-sampling of neighboring cells by the same ablation mark can mix signals, though the paper reports this only minimally affects co-culture cell-type prediction accuracy.
- There is a current lack of established validation approaches in single-cell metabolomics, so feature-subset reproducibility across datasets is not standardized.
- The published feature subset is dataset-specific; transferring the 88-metabolite list to other cell systems is not validated by this paper.

## Evidence
