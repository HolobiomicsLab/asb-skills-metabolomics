---
name: single-cell-quality-control-filtering
description: Use when single-cell readouts derived from SpaceM ablation-mark aggregation
  (h5ad/anndata objects) need QC prior to correlation, isotopologue, or uptake-fraction
  analysis, to remove poorly segmented or low-signal cells.
when_to_use_negative:
- Input is raw imzML or SpaceM spacem_data output that has not yet passed through
  sc_analysis.py — QC is applied after single-cell analysis and aggregation, not at
  the ion-image stage
- Analysis is per-pixel tissue imaging (mouse brain experiments), where the workflow
  fits binomial distributions per pixel instead of filtering single cells
- Cells have already been QC-filtered downstream (e.g. the shared per-cell anndata
  object already reflects published QC filtering)
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0219
tools:
- name: 13C-SpaceM
  role: Provides the single-cell isotope tracing pipeline whose QC and filtering step
    this skill implements
  repo: https://github.com/Buglakova/13C-SpaceM
- name: Scanpy
  role: Used to load and manipulate the pickled anndata single-cell objects on which
    QC filtering is performed
  repo: https://github.com/Buglakova/13C-SpaceM
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s42255-024-01118-4
    title: Spatial single-cell isotope tracing reveals heterogeneity of de novo fatty
      acid synthesis in cancer
  source_content_hash: sha256:7bdd24c9f79dfe1765af3c240d78ab0f3852821d2b85728a18c54ee6d90d1d85
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Buglakova/13C-SpaceM
  tools:
  - 13C-SpaceM
  - Scanpy
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0219
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 4
---

# single-cell-quality-control-filtering

## Summary

Quality-control step of the 13C-SpaceM single-cell isotope tracing workflow: after aggregating per-slide anndata objects, cells are filtered by cell size, number of ablation marks (detected ions) and total detected ion intensity before downstream analysis or figure generation.

## When to use

Use when single-cell readouts derived from SpaceM ablation-mark aggregation (h5ad/anndata objects) need QC prior to correlation, isotopologue, or uptake-fraction analysis, to remove poorly segmented or low-signal cells.

## When NOT to use

- Input is raw imzML or SpaceM spacem_data output that has not yet passed through sc_analysis.py — QC is applied after single-cell analysis and aggregation, not at the ion-image stage
- Analysis is per-pixel tissue imaging (mouse brain experiments), where the workflow fits binomial distributions per pixel instead of filtering single cells
- Cells have already been QC-filtered downstream (e.g. the shared per-cell anndata object already reflects published QC filtering)

## Inputs

- Per-slide anndata (h5ad) objects produced by sciso/sc_analysis.py (e.g. $data_dir/20220322_AB_DKFZHypoxia/slide2/anndata)
- Per-cell covariates: cell size, number of detected ions, total detected ion intensity
- Normalized GFP intensity values from microscopy images for cell classification

## Outputs

- QC-filtered, aggregated anndata object containing all per-cell information (normalized isotopologue distributions, relative uptake fractions) used for manuscript plots
- Cell condition labels (e.g. normoxic/hypoxic) from normalized GFP intensity classification

## How to apply

After running sc_analysis.py per slide (which performs natural isotope abundance correction, median aggregation of ablation-mark signal per cell, and normalization of isotopologue distributions), aggregate the per-slide anndata objects together and apply QC filtering on three cell-level covariates: cell size, number of detected ions (ablation marks), and total detected ion intensity. Subsequently classify cells using normalized GFP intensity to separate conditions (e.g. normoxic vs hypoxic populations). The filtered, classified anndata objects are the inputs used to generate manuscript figures (Figures 2-4 are based on this single-cell data). Rationale: unfiltered cells can carry segmentation artifacts or weak MALDI signal that distort per-cell relative uptake estimates such as the palmitate M+0 fraction used to separate hypoxic from normoxic cells.

## Related tools

- **13C-SpaceM** (Provides the single-cell isotope tracing pipeline whose QC and filtering step this skill implements) — https://github.com/Buglakova/13C-SpaceM
- **Scanpy** (Used to load and manipulate the pickled anndata single-cell objects on which QC filtering is performed) — https://github.com/Buglakova/13C-SpaceM

## Examples

```
python sciso/sc_analysis.py $data_dir/20220322_AB_DKFZHypoxia/slide2/spacem_data $data_dir/20220322_AB_DKFZHypoxia/slide2/anndata
```

## Evaluation signals

- QC'd control (normoxic) cells reproduce the manuscript correlation pattern: myristate relative uptake uncorrelated with other fatty acids, while palmitate, palmitoleate, stearate and oleate are strongly positively correlated
- Filtered populations show the expected separation: hypoxic and normoxic cells clearly separate on the palmitate M+0 fraction
- No cells remain with near-zero total detected ion intensity or degenerate cell sizes after filtering
- Resulting anndata object loads cleanly with Scanpy across pandas/anndata/pickle/scanpy version combinations (check environment_dump.yml if errors)

## Limitations

- Binomial fitting to estimate uptake fraction is unreliable when overall substrate usage is very low (e.g. hypoxia/normoxia experiment); it was only used for ACLYkd data
- MALDI-AIF only samples fatty acids from lipids ionized in negative mode (primarily acidic glycerophospholipids), so QC'd cells still reflect only a subset of the lipidome
- Scanpy pickled anndata objects can fail to read with mismatched pandas/anndata/pickle/scanpy versions; use the environment_dump.yml as reference

## Evidence

- [readme] After single-cell analysis data from all slides is aggregated together and quality control is performed, filtering out cells by size and number and total intensity of detected ions.: "After single-cell analysis data from all slides is aggregated together and quality control is performed, filtering out cells by size and number and total intensity of detected ions."
- [methods] filtering out cells by size and number and total intensity of detected ions: "filtering out cells by size and number and total intensity of detected ions"
- [readme] Subsequently the cells are classified using normalized GFP intensity.: "Subsequently the cells are classified using normalized GFP intensity."
- [readme] This project uses Scanpy's pickled `anndata` objects to store the single-cell data.: "This project uses Scanpy's pickled `anndata` objects to store the single-cell data."
