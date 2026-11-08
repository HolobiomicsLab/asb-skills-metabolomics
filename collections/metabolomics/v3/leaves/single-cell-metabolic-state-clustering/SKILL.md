---
name: single-cell-metabolic-state-clustering
description: Use when a per-cell spatio-molecular matrix (normalized single-cell metabolomes
  with microscopy-derived phenotypic properties) needs to be partitioned into metabolic
  states, or when assessing whether such state assignments are robust to cell-filtering
  choices such as excluding co-sampling ablation.
when_to_use_negative:
- The goal is cell-type classification rather than state discovery — in that case
  use the co-culture cell-type classifier (91.3% accuracy on HeLa/NIH3T3 single-cell
  metabolomes) instead of state clustering
- Input is raw MALDI-imaging (.RAW/.imzML/.ibd) or unsegmented microscopy tiles rather
  than the per-cell matrix MORPHnMOL.csv — first run the SpaceM pipeline (runAnalysis.py)
  to generate the matrix
- Sample-level or bulk metabolomics data — the skill operates on single-cell-level
  metabolome measurements and cannot recover cell-cell heterogeneity from bulk profiles
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_2229
- http://edamontology.org/topic_3520
tools:
- name: SCANPY
  role: Performs the metabolic-state clustering of single-cell metabolomes and reproduces
    original state assignments on the filtered cell set
- name: Python
  role: Runs the analysis (load MORPHnMOL.csv with pandas, filter co-sampled cells,
    invoke SCANPY, write comparison CSV)
  repo: github.com/alexandrovteam/SpaceM
- name: SpaceM
  role: Generates the spatio-molecular matrix (MORPHnMOL.csv) that serves as the clustering
    input
  repo: github.com/alexandrovteam/SpaceM
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41592-021-01198-0
    title: SpaceM reveals metabolic states of single cells
  source_content_hash: sha256:3e1cbcbe9b56ff1644e368c3e196e5ad235f5565d284dc7b088deda7d1149074
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Python
  - SCANPY
  - SpaceM
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_2229
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# single-cell-metabolic-state-clustering

## Summary

Clustering per-cell metabolic profiles into discrete metabolic states using SCANPY on SpaceM's spatio-molecular matrix (MORPHnMOL.csv), as applied to dHepaRG hepatocytes in the NAFLD/NASH model and validated via co-sampling ablation-mark filtering.

## When to use

Use when a per-cell spatio-molecular matrix (normalized single-cell metabolomes with microscopy-derived phenotypic properties) needs to be partitioned into metabolic states, or when assessing whether such state assignments are robust to cell-filtering choices such as excluding co-sampling ablation marks.

## When NOT to use

- The goal is cell-type classification rather than state discovery — in that case use the co-culture cell-type classifier (91.3% accuracy on HeLa/NIH3T3 single-cell metabolomes) instead of state clustering
- Input is raw MALDI-imaging (.RAW/.imzML/.ibd) or unsegmented microscopy tiles rather than the per-cell matrix MORPHnMOL.csv — first run the SpaceM pipeline (runAnalysis.py) to generate the matrix
- Sample-level or bulk metabolomics data — the skill operates on single-cell-level metabolome measurements and cannot recover cell-cell heterogeneity from bulk profiles

## Inputs

- Per-cell spatio-molecular matrix MORPHnMOL.csv from the scAnalysis sub-folder (normalized metabolic profiles plus microscopy-derived fluorescence intensities and morpho-spatial features)
- Co-sampling annotation per cell (whether the ablation mark co-sampled a neighboring cell), for the filtered variant of the analysis
- Culture-condition labels for dHepaRG hepatocytes (control, fatty-acid stimulation, IL-17A, NF-κB inhibition) for per-condition state-fraction comparison

## Outputs

- Metabolic-state assignments per cell (SCANPY clustering)
- Per-condition metabolic-state fractions for filtered vs. unfiltered runs
- Metabolic-state markers with significance statistics (expected significance drop after filtering)
- Quantified comparison table such as cosampling_filtered_state_comparison.csv (cell counts, state fractions, marker stability, classification accuracy)

## How to apply

Load the unfiltered per-cell spatio-molecular matrix MORPHnMOL.csv from the scAnalysis folder with pandas in Python, then run metabolic-state clustering of the single-cell metabolomes in SCANPY (using the microscopy-derived fluorescence/morpho-spatial properties as phenotypic context). When robustness is the question, apply the sampling-specificity filter by dropping all cells whose ablation mark co-sampled a neighboring cell — in the paper this reduced the hepatocyte set from 29,738 to 21,009 cells — and re-run the clustering to compare state assignments. Judge the result by three invariants: the same metabolic states are recovered (Supp. Fig. 11A), the balance of states within each culture condition (e.g. fatty-acid stimulation, IL-17A inflammation, NF-κB inhibition) is unchanged, and metabolic-state markers remain the same even though their statistical significance may drop due to the lower cell count. Note that intermediate-state cells can appear bimodal — the paper leaves open whether this is a clustering artifact or biological heterogeneity of lipid-drop (LD) compositions.

## Related tools

- **SCANPY** (Performs the metabolic-state clustering of single-cell metabolomes and reproduces original state assignments on the filtered cell set)
- **Python** (Runs the analysis (load MORPHnMOL.csv with pandas, filter co-sampled cells, invoke SCANPY, write comparison CSV)) — github.com/alexandrovteam/SpaceM
- **SpaceM** (Generates the spatio-molecular matrix (MORPHnMOL.csv) that serves as the clustering input) — github.com/alexandrovteam/SpaceM

## Examples

```
python runAnalysis.py  # then in downstream analysis: import pandas as pd; df = pd.read_csv('Analysis/scAnalysis/MORPHnMOL.csv')
```

## Evaluation signals

- Filtered run recovers the same metabolic states as the unfiltered run (Supp. Fig. 11A)
- Balance of metabolic states within each hepatocyte culture condition is unchanged after excluding co-sampling ablation marks
- Metabolic-state markers are identical between runs; only their statistical significance drops, attributable to the lower cell number (21,009 vs. 29,738)
- Cell-type classification accuracy on co-culture metabolomes is near-unchanged (91.3% reported) after filtering (Supp. Fig. 6)
- Cell counts of the filtered hepatocyte set match the expected 21,009 of 29,738 cells

## Limitations

- Excluding co-sampling marks reduces the number of cells, lowering statistical significance of state markers without changing the markers themselves
- Bimodality of intermediate-state cells is unresolved — could be a clustering artifact or genuine biological heterogeneity of LD compositions
- Current lack of established validation approaches in single-cell metabolomics limits independent verification of state assignments

## Evidence
