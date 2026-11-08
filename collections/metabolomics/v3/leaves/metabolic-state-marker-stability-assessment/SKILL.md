---
name: metabolic-state-marker-stability-assessment
description: Use when a SpaceM spatio-molecular matrix (MORPHnMOL.csv) contains ablation
  marks that co-sampled neighboring cells and you must quantify whether excluding
  them changes metabolic-state clustering, marker significance, or cell type classification
  accuracy.
when_to_use_negative:
- Input is not a SpaceM-derived per-cell spatio-molecular matrix (MORPHnMOL.csv) but
  raw MALDI imaging data (.RAW/.imzML) or pre-stitched microscopy — the skill operates
  downstream of SpaceM's runAnalysis.py output
- The question is method development of segmentation, tile stitching, or METASPACE
  annotation rather than robustness of downstream clustering/classification to co-sampling
- The dataset has no spatially-heterogeneous co-culture or neighboring-cell co-sampling,
  so the sampling-specificity filter has nothing to remove
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: SpaceM
  role: Source of the per-cell spatio-molecular matrix (MORPHnMOL.csv) and the co-sampling
    ablation-mark annotations that the filtering skill consumes
  repo: github.com/alexandrovteam/SpaceM
- name: SCANPY
  role: Re-runs metabolic-state clustering on the filtered single-cell metabolomes
    to check state recovery
  repo: github.com/alexandrovteam/SpaceM
- name: Python
  role: pandas-based filtering of co-sampled cells, classifier re-training/evaluation,
    and writing of the comparison CSV
  repo: github.com/alexandrovteam/SpaceM
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41592-021-01198-0
    title: SpaceM reveals metabolic states of single cells
  source_content_hash: sha256:a3af77c37edfc44b71201bd4d74408ac558715498902166cc17ef8c6ee3fe240
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
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# metabolic-state-marker-stability-assessment

## Summary

Re-runs SpaceM downstream analysis after dropping co-sampling ablation marks (and diffused metabolites) to test whether cell type classification, metabolic-state assignments, and state balance per condition are robust to ablation-mark cross-contamination in the HeLa/NIH3T3 co-culture and dHepaRG NAFLD model.

## When to use

Use when a SpaceM spatio-molecular matrix (MORPHnMOL.csv) contains ablation marks that co-sampled neighboring cells and you must quantify whether excluding them changes metabolic-state clustering, marker significance, or cell type classification accuracy.

## When NOT to use

- Input is not a SpaceM-derived per-cell spatio-molecular matrix (MORPHnMOL.csv) but raw MALDI imaging data (.RAW/.imzML) or pre-stitched microscopy — the skill operates downstream of SpaceM's runAnalysis.py output
- The question is method development of segmentation, tile stitching, or METASPACE annotation rather than robustness of downstream clustering/classification to co-sampling
- The dataset has no spatially-heterogeneous co-culture or neighboring-cell co-sampling, so the sampling-specificity filter has nothing to remove

## Inputs

- Unfiltered per-cell spatio-molecular matrix MORPHnMOL.csv from the Analysis/scAnalysis sub-folder
- Per-cell ablation-mark co-sampling annotation (which ablation marks hit parts of a neighboring cell)
- Co-culture condition labels (HeLa/NIH3T3) and hepatocyte culture condition metadata (fatty acid stimulation, IL-17A inflammation, NF-κB inhibition)

## Outputs

- Filtered single-cell metabolome set (21,009 cells) vs unfiltered set (29,738 cells)
- cosampling_filtered_state_comparison.csv with cell counts, state fractions per condition, marker stability, and classification accuracy
- Recovered metabolic-state assignments for the filtered hepatocytes (Supp. Fig. 11A)
- Cell-type classification accuracy for the filtered co-culture metabolomes (Supp. Fig. 6)

## How to apply

Load the unfiltered per-cell spatio-molecular matrix MORPHnMOL.csv from the Analysis/scAnalysis sub-folder with pandas. Apply the sampling-specificity filter by dropping all cells whose assigned ablation mark co-sampled a neighboring cell, yielding the filtered hepatocyte set (21,009 cells) alongside the unfiltered set (29,738 cells). Re-run metabolic-state clustering of the filtered hepatocytes in SCANPY and check that the same states are recovered (Supp. Fig. 11A). Re-train and evaluate the co-culture cell-type classifier on the filtered single-cell metabolomes to quantify the change in classification accuracy (Supp. Fig. 6; the unfiltered accuracy is 91.3%). Finally compare state fractions per culture condition and re-assess metabolic-state marker significance, which is expected to drop with the lower cell number even if markers themselves do not change, and write the comparison to cosampling_filtered_state_comparison.csv.

## Related tools

- **SpaceM** (Source of the per-cell spatio-molecular matrix (MORPHnMOL.csv) and the co-sampling ablation-mark annotations that the filtering skill consumes) — github.com/alexandrovteam/SpaceM
- **SCANPY** (Re-runs metabolic-state clustering on the filtered single-cell metabolomes to check state recovery) — github.com/alexandrovteam/SpaceM
- **Python** (pandas-based filtering of co-sampled cells, classifier re-training/evaluation, and writing of the comparison CSV) — github.com/alexandrovteam/SpaceM

## Examples

```
import pandas as pd; df = pd.read_csv('MF/Analysis/scAnalysis/MORPHnMOL.csv')
```

## Evaluation signals

- Filtered run recovers the same metabolic states as the unfiltered run (Supp. Fig. 11A reproduction)
- Cell-type classification accuracy remains near 91.3% after excluding co-sampling marks (Supp. Fig. 6)
- Balance of metabolic states within each hepatocyte culture condition is unchanged between filtered and unfiltered runs
- Cell-count check: filtered set ≈ 21,009 hepatocytes out of 29,738 unfiltered
- Marker identity stable, but marker statistical significance is expected to drop with the reduced cell number — flag a result where significance increases as suspicious

## Limitations

- Excluding co-sampling ablation marks reduces the statistical significance of metabolic-state markers, likely due to the lower number of cells
- Bimodality of intermediate-state cells could reflect a clustering artifact or biological heterogeneity of LD compositions, so state-level conclusions may be ambiguous
- Requires ablation marks visible and non-overlapping in post-MALDI microscopy, and SpaceM currently requires tile stitching optimized for Nikon Ti-E output
- Co-sampling assessment is specific to spatially-heterogeneous co-cultures; it minimally affects cell-type prediction only in that setting

## Evidence
