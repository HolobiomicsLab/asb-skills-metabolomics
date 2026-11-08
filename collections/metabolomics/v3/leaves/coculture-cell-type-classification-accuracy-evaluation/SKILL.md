---
name: coculture-cell-type-classification-accuracy-evaluation
description: Use when a SpaceM spatio-molecular matrix from a spatially-heterogeneous
  co-culture needs cell-type prediction from metabolic profiles, or when assessing
  the effect of filtering co-sampling ablation marks on classification accuracy.
when_to_use_negative:
- Input is not a SpaceM single-cell metabolome matrix (e.g. bulk metabolomics or single-cell
  transcriptomics) — classifier performance on other modalities is not established
  by this paper
- The culture is isogenic or spatially homogeneous, so there is no spatially-heterogeneous
  co-culture in which co-sampling of neighboring cells could confound the labels
- Post-MALDI microscopy lacks visible, non-overlapping ablation marks, so cells cannot
  be reliably filtered or assigned
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: SpaceM
  role: Produces the per-cell spatio-molecular matrix (MORPHnMOL.csv) whose metabolome
    columns feed the co-culture classifier
  repo: github.com/alexandrovteam/SpaceM
- name: Python
  role: Loads MORPHnMOL.csv with pandas, trains/evaluates the cell-type classifier,
    and writes the filtered-vs-unfiltered comparison table
- name: SCANPY
  role: Re-runs metabolic-state clustering on the filtered hepatocyte set to confirm
    the same state assignments after co-sampling filtering
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41592-021-01198-0
    title: SpaceM reveals metabolic states of single cells
  source_content_hash: sha256:def43a775892c9d4a22b6dc908551e79d9c30a8323b22119a9932d04690ac33f
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Python
  - SCANPY
  - SpaceM
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# coculture-cell-type-classification-accuracy-evaluation

## Summary

Trains and evaluates a classifier that distinguishes co-cultured cell types solely from SpaceM single-cell metabolomes (HeLa/NIH3T3 benchmark accuracy 91.3%), and quantifies whether removing co-sampling ablation marks and diffused metabolites degrades that accuracy.

## When to use

Use when a SpaceM spatio-molecular matrix from a spatially-heterogeneous co-culture needs cell-type prediction from metabolic profiles, or when assessing the effect of filtering co-sampling ablation marks on classification accuracy.

## When NOT to use

- Input is not a SpaceM single-cell metabolome matrix (e.g. bulk metabolomics or single-cell transcriptomics) — classifier performance on other modalities is not established by this paper
- The culture is isogenic or spatially homogeneous, so there is no spatially-heterogeneous co-culture in which co-sampling of neighboring cells could confound the labels
- Post-MALDI microscopy lacks visible, non-overlapping ablation marks, so cells cannot be reliably filtered or assigned

## Inputs

- Per-cell spatio-molecular matrix MORPHnMOL.csv (normalized metabolite intensities + microscopy-derived morpho-spatial features) from MF/Analysis/scAnalysis
- Cell-type labels for the co-culture (e.g. HeLa/NIH3T3) derived from microscopy fluorescence intensities
- Ablation-mark co-sampling annotations per cell (for the filtered run)

## Outputs

- Cell-type classification accuracy for the co-culture (benchmark: 91.3% HeLa/NIH3T3)
- cosampling_filtered_state_comparison.csv with cell counts, state fractions per condition, marker stability, and classification accuracy
- Re-produced metabolic-state assignments (Supp. Fig. 11A) from the filtered hepatocyte set clustered in SCANPY

## How to apply

Load the unfiltered per-cell spatio-molecular matrix MORPHnMOL.csv from the scAnalysis sub-folder with pandas in Python; it contains a normalized metabolic profile plus microscopy-derived phenotypic properties (fluorescence intensities, morpho-spatial features) per cell. Train a cell-type classifier on the single-cell metabolomes and evaluate accuracy — the paper reports 91.3% for HeLa/NIH3T3 co-culture prediction solely from metabolomes. For the ablation-robustness variant, drop all cells whose assigned ablation mark co-sampled a neighboring cell (yielding the filtered set, e.g. 21,009 vs 29,738 hepatocyte cells), discard diffused metabolites, re-train and re-evaluate, and compare accuracy, state fractions per culture condition, and metabolic-state marker significance between filtered and unfiltered runs. The expected outcome, grounded in the paper, is near-unchanged classification accuracy and unchanged balance of metabolic states per condition, with marker statistical significance reduced only because of the lower cell number. Write the quantified comparison (cell counts, state fractions, marker stability, classification accuracy) to cosampling_filtered_state_comparison.csv.

## Related tools

- **SpaceM** (Produces the per-cell spatio-molecular matrix (MORPHnMOL.csv) whose metabolome columns feed the co-culture classifier) — github.com/alexandrovteam/SpaceM
- **Python** (Loads MORPHnMOL.csv with pandas, trains/evaluates the cell-type classifier, and writes the filtered-vs-unfiltered comparison table)
- **SCANPY** (Re-runs metabolic-state clustering on the filtered hepatocyte set to confirm the same state assignments after co-sampling filtering)

## Examples

```
import pandas as pd; df = pd.read_csv('MF/Analysis/scAnalysis/MORPHnMOL.csv')
```

## Evaluation signals

- Unfiltered HeLa/NIH3T3 classification accuracy reproduces the reported 91.3%
- Filtered run (co-sampling marks removed) yields near-unchanged classification accuracy relative to the unfiltered run
- Metabolic states identified in the filtered hepatocyte set match the original assignments (Supp. Fig. 11A reproduction)
- Balance of metabolic states within each culture condition is unchanged after filtering
- State-marker statistical significance drops in the filtered run, consistent with the lower cell number (e.g. 21,009 vs 29,738 cells), while the marker identities are unchanged

## Limitations

- Co-sampling only minimally affects cell-type prediction, but excluding co-sampling ablation marks reduces statistical significance of metabolic-state markers, likely due to the lower number of cells
- Requires ablation marks to be visible and non-overlapping in post-MALDI microscopy; overlapping marks break cell assignment
- The field lacks established validation approaches in single-cell metabolomics, so accuracy claims rest on self-consistency benchmarks like the 91.3% co-culture prediction

## Evidence
