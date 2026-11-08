---
name: sampling-area-overlap-normalization-to-per-cell-molecular-assignment
description: Use when MALDI-imaging MS data (.imzML/.ibd/.RAW/.UDP) from a laser-ablation
  experiment on cultured cells must be converted into per-cell molecular profiles,
  i.e. when ablation marks must be linked to segmented cells to build a spatio-molecular
  matrix.
when_to_use_negative:
- Ablation marks are overlapping or not visible in the post-MALDI microscopy — mark
  segmentation and per-cell assignment are unreliable
- Input is already a per-cell feature/molecular table (e.g. scRNA-seq or a finished
  MORPHnMOL.csv) — this skill addresses the raw-image-to-cell intensity assignment
  step only
- Tissue-section rather than cultured-cell MALDI-IMS experiments, where tiled cell-culture
  microscopy and cell segmentation do not apply
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: SpaceM
  role: Implements the mark-to-cell co-registration and sampling-area-overlap normalization
    pipeline; run via runAnalysis.py
  repo: github.com/alexandrovteam/SpaceM
- name: CellProfiler
  role: Segments cells in the stitched pre-MALDI image using the CP pipeline and project
    files referenced in paths.json
- name: METASPACE
  role: Analyzes the MALDI-IMS data to obtain per-mark molecular profiles that are
    joined to co-registered ablation marks
- name: Fiji
  role: Used for image processing steps; its executable path must be registered in
    paths.json
- name: Nikon Ti-E
  role: Microscope whose tile output format the stitching code is optimized for (out.txt
    coordinates)
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41592-021-01198-0
    title: SpaceM reveals metabolic states of single cells
  source_content_hash: sha256:92c52047714f35a8207541857cac3be029295f569761cc8fee7c42c939d4e299
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - CellProfiler
  - Fiji
  - METASPACE
  - Nikon Ti-E
  - Python
  - SpaceM
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7692307692307693
n_claims: 13
---

# Sampling-area overlap normalization to per-cell molecular assignment

## Summary

SpaceM assigns MALDI-IMS metabolite intensities to individual cultured cells by co-registering laser-ablation marks detected in post-MALDI microscopy with CellProfiler-segmented cells from pre-MALDI microscopy, and normalizing per-mark intensities by the 45% sampling-area overlap so each cell receives only the fraction of signal from the cell area actually ablated.

## When to use

Use when MALDI-imaging MS data (.imzML/.ibd/.RAW/.UDP) from a laser-ablation experiment on cultured cells must be converted into per-cell molecular profiles, i.e. when ablation marks must be linked to segmented cells to build a spatio-molecular matrix.

## When NOT to use

- Ablation marks are overlapping or not visible in the post-MALDI microscopy — mark segmentation and per-cell assignment are unreliable
- Input is already a per-cell feature/molecular table (e.g. scRNA-seq or a finished MORPHnMOL.csv) — this skill addresses the raw-image-to-cell intensity assignment step only
- Tissue-section rather than cultured-cell MALDI-IMS experiments, where tiled cell-culture microscopy and cell segmentation do not apply

## Inputs

- Tiled pre-MALDI microscopy images (tile_1.tif, tile_2.tif, ...) with pen-mark fiducials
- Tiled post-MALDI microscopy images showing visible, non-overlapping ablation marks
- out.txt text file with upper-left pixel x-y coordinates (um) of each tile
- MALDI-IMS raw data: *.RAW, *.UDP, *.imzML, *.ibd files
- CellProfiler pipeline and project files able to segment the cultured cells
- paths.json registering MF, CellProfiler, and Fiji paths

## Outputs

- Co-registered ablation-mark-to-cell assignment map
- Per-cell normalized metabolic profiles joined to microscopy-derived phenotypic properties (fluorescence intensities, morpho-spatial features)
- Final spatio-molecular matrix MORPHnMOL.csv in MF/Analysis/scAnalysis

## How to apply

Acquire tiled pre- and post-MALDI microscopy images with black pen-mark fiducials, stitch them using out.txt upper-left pixel x-y coordinates (in um; code optimized for Nikon Ti-E output), and segment cells in the stitched pre-MALDI image with the CellProfiler pipeline/project registered in paths.json. Detect the MALDI laser ablation marks in the post-MALDI image and co-register them with segmented cells at subcellular precision to estimate which cell parts were locally ablated — this co-registration is the cornerstone of the method. Analyze the MALDI data in METASPACE to obtain per-mark molecular profiles, then apply the 45% sampling-area-overlap normalization when joining marks to cells, so intensities are attributed according to how much of each cell's area was sampled by each mark. Execute `python runAnalysis.py` to produce the final spatio-molecular matrix; co-sampling of neighboring cells by one mark is tolerated and only minimally affects cell-type prediction accuracy, but marks that sample multiple cell types should be flagged and optionally excluded.

## Related tools

- **SpaceM** (Implements the mark-to-cell co-registration and sampling-area-overlap normalization pipeline; run via runAnalysis.py) — github.com/alexandrovteam/SpaceM
- **CellProfiler** (Segments cells in the stitched pre-MALDI image using the CP pipeline and project files referenced in paths.json)
- **METASPACE** (Analyzes the MALDI-IMS data to obtain per-mark molecular profiles that are joined to co-registered ablation marks)
- **Fiji** (Used for image processing steps; its executable path must be registered in paths.json)
- **Nikon Ti-E** (Microscope whose tile output format the stitching code is optimized for (out.txt coordinates))

## Examples

```
python runAnalysis.py
```

## Evaluation signals

- Pipeline completes and MORPHnMOL.csv is written inside the scAnalysis sub-folder of MF/Analysis
- Every row of the matrix contains both a normalized metabolic profile and microscopy-derived phenotypic properties for a segmented cell
- Post-MALDI ablation marks are visible and non-overlapping before assignment (data requirement)
- Per-mark intensities are scaled by the sampling-area overlap (45% normalization rule) so co-sampled neighbor signal is proportionally attributed
- Cell-type prediction from the resulting single-cell metabolomes reaches ~91.3% accuracy in the mixed-co-culture benchmark, and excluding co-sampling marks yields the same metabolic states

## Limitations

- Co-sampling of neighboring cells by the same ablation mark mixes signals; excluding such marks reduces statistical significance of metabolic state markers due to fewer cells
- Stitching code is optimized for the Nikon Ti-E output format; pre-stitched images are only accepted in later versions
- Requires ablation marks to be visible and non-overlapping in post-MALDI microscopy
- No version numbers are given for CellProfiler, Fiji, or Python (only CellProfiler 3.0.0 and a specific 2015 Fiji release are named for installation)

## Evidence

- [methods] 45% normalization step: [evidence span withheld — anchor status: not_found]
