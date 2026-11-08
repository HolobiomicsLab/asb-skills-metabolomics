---
name: co-sampling-ablation-mark-exclusion
description: Use when validating SpaceM results in spatially-heterogeneous co-cultures
  or clustered cultures by testing whether removing co-sampled ablation marks changes
  cell-type classification accuracy or metabolic-state assignments.
when_to_use_negative:
- The goal is routine SpaceM cell-type classification from a fresh co-culture — co-sampling
  only minimally affects prediction accuracy, so the filter is a validation exercise,
  not a required preprocessing step.
- Input is raw MALDI (.RAW/.imzML) or microscopy data — this skill operates downstream
  on the already-generated MORPHnMOL.csv matrix.
- The culture is a sparse monolayer where ablation marks never overlap neighboring
  cells, making the filter a no-op.
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3518
- http://edamontology.org/topic_0092
tools:
- name: SpaceM
  role: Produces the MORPHnMOL.csv spatio-molecular matrix that this filter operates
    on, including co-registration of ablation marks to cells
  repo: https://github.com/alexandrovteam/SpaceM
- name: Python
  role: Loads MORPHnMOL.csv with pandas, applies the co-sampling filter, and writes
    the comparison CSV
- name: SCANPY
  role: Re-runs metabolic-state clustering of the filtered hepatocyte set to compare
    against original state assignments
- name: CellProfiler
  role: Upstream segmentation of cells from microscopy so ablation marks can be assigned
    to cells
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41592-021-01198-0
    title: SpaceM reveals metabolic states of single cells
  source_content_hash: sha256:94f90e1bdea0faed9aebe844b92d35dee01188c48ebfd086e6bd1e877e25e12a
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/alexandrovteam/SpaceM
  tools:
  - CellProfiler
  - Python
  - SCANPY
  - SpaceM
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3518
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# co-sampling-ablation-mark-exclusion

## Summary

A validation procedure for SpaceM that filters the per-cell spatio-molecular matrix (MORPHnMOL.csv) to remove cells whose ablation mark co-sampled a neighboring cell (and discards diffused metabolites), then re-runs metabolic-state clustering and cell-type classification to quantify the impact of co-sampling on downstream results.

## When to use

Use when validating SpaceM results in spatially-heterogeneous co-cultures or clustered cultures by testing whether removing co-sampled ablation marks changes cell-type classification accuracy or metabolic-state assignments.

## When NOT to use

- The goal is routine SpaceM cell-type classification from a fresh co-culture — co-sampling only minimally affects prediction accuracy, so the filter is a validation exercise, not a required preprocessing step.
- Input is raw MALDI (.RAW/.imzML) or microscopy data — this skill operates downstream on the already-generated MORPHnMOL.csv matrix.
- The culture is a sparse monolayer where ablation marks never overlap neighboring cells, making the filter a no-op.

## Inputs

- Unfiltered per-cell spatio-molecular matrix MORPHnMOL.csv (located in the MF/Analysis/scAnalysis sub-folder)
- Per-cell ablation-mark co-sampling annotations (cells whose ablation mark co-sampled a neighboring cell)
- Original metabolic-state assignments and cell-type classifier training data for the filtered vs. unfiltered comparison

## Outputs

- Filtered hepatocyte cell set (e.g. 21,009 cells) alongside the unfiltered set (e.g. 29,738 cells)
- cosampling_filtered_state_comparison.csv with cell counts, state fractions, marker stability, and classification accuracy
- Reproduced metabolic-state assignments on the filtered set (Supp. Fig. 11A) and re-evaluated cell-type classification accuracy (Supp. Fig. 6)

## How to apply

Load the unfiltered per-cell spatio-molecular matrix MORPHnMOL.csv from the scAnalysis folder with pandas in Python, then apply the sampling-specificity filter by dropping all cells whose assigned ablation mark co-sampled a neighboring cell (e.g. 29,738 unfiltered hepatocytes reduce to 21,009 filtered cells). Re-run metabolic-state clustering of the filtered hepatocytes in SCANPY to check whether the same states are recovered, and re-train and evaluate the co-culture cell-type classifier on the filtered single-cell metabolomes to quantify the change in classification accuracy (the HeLa/NIH3T3 co-culture accuracy stays at 91.3%). Compare state fractions per culture condition and re-assess metabolic-state marker significance, expecting marker significance to drop due to the lower cell number even though state balance per condition is preserved. Write the quantified comparison — cell counts, state fractions, marker stability, and classification accuracy — to cosampling_filtered_state_comparison.csv.

## Related tools

- **SpaceM** (Produces the MORPHnMOL.csv spatio-molecular matrix that this filter operates on, including co-registration of ablation marks to cells) — https://github.com/alexandrovteam/SpaceM
- **Python** (Loads MORPHnMOL.csv with pandas, applies the co-sampling filter, and writes the comparison CSV)
- **SCANPY** (Re-runs metabolic-state clustering of the filtered hepatocyte set to compare against original state assignments)
- **CellProfiler** (Upstream segmentation of cells from microscopy so ablation marks can be assigned to cells)

## Examples

```
python -c "import pandas as pd; df = pd.read_csv('MF/Analysis/scAnalysis/MORPHnMOL.csv'); df_f = df[~df['co_sampled']]; df_f.to_csv('cosampling_filtered_state_comparison.csv')"
```

## Evaluation signals

- Filtered run recovers the same metabolic states as the unfiltered run (Supp. Fig. 11A)
- Cell-type classification accuracy on the filtered metabolomes remains near 91.3% in the HeLa/NIH3T3 co-culture (Supp. Fig. 6)
- Balance of metabolic states within each hepatocyte culture condition is unchanged between filtered and unfiltered runs
- Metabolic-state markers remain the same but their statistical significance drops, consistent with the reduced cell count (29,738 → 21,009 cells)
- Output CSV contains quantified cell counts, state fractions, marker stability, and classification accuracy for both filtered and unfiltered sets

## Limitations

- Excluding co-sampling marks reduces statistical significance of metabolic-state markers, likely due to the lower number of cells (21,009 vs 29,738 hepatocytes).
- The filter discards diffused metabolites along with co-sampled cells, removing data points that could otherwise inform downstream statistics.
- Bimodality of intermediate-state cells could reflect a clustering artifact or biological heterogeneity of lipid-droplet (LD) compositions, and the filter does not resolve this ambiguity.

## Evidence

- [methods] task workflow: [evidence span withheld — anchor status: not_found]
- [methods] filter definition: [evidence span withheld — anchor status: not_found]
