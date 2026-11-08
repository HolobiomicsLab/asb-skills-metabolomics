---
name: cellprofiler-cell-segmentation-pipeline-configuration
description: Use when running the SpaceM pipeline on MALDI-imaging data (*.RAW, *.UDP,
  *.imzML, *.ibd) plus pre/post-MALDI tiled microscopy, and per-cell segmentation
  via a CellProfiler pipeline must be configured in paths.json.
when_to_use_negative:
- Data is not SpaceM-format tiled microscopy paired with MALDI-imaging files — the
  CP pipeline/project paths are consumed specifically by runAnalysis.py
- Images are already single pre-stitched fields without tiles and out.txt coordinates
  — the Nikon Ti-E tile-stitching workflow does not apply
edam_operation: http://edamontology.org/operation_3443
edam_topics:
- http://edamontology.org/topic_3382
- http://edamontology.org/topic_3676
tools:
- name: CellProfiler
  role: Provides the segmentation software; the CP pipeline and CP project files that
    segment cells are registered in paths.json and invoked by SpaceM during runAnalysis.py
  repo: https://cellprofiler.org/previous_releases/
- name: SpaceM
  role: Main pipeline that stitches tiles, applies the CellProfiler segmentation,
    and produces the spatio-molecular matrix MORPHnMOL.csv
  repo: https://github.com/alexandrovteam/SpaceM
- name: Python
  role: Execution environment; runAnalysis.py prompts the user through stitching and
    CellProfiler segmentation steps
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41592-021-01198-0
    title: SpaceM reveals metabolic states of single cells
  source_content_hash: sha256:495cd8a3f41fff741fb84f4253a6c0ce37479055a541a07a9944fc3e8ee71596
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - CellProfiler
  - Fiji
  - Python
  - SpaceM
  edam_operation: http://edamontology.org/operation_3443
  edam_topics:
  - http://edamontology.org/topic_3382
  - http://edamontology.org/topic_3676
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# cellprofiler-cell-segmentation-pipeline-configuration

## Summary

Configure and register the CellProfiler segmentation pipeline and project files that SpaceM uses to segment cultured cells from tiled microscopy images, so metabolite intensities from MALDI ablation marks can be assigned to individual cells.

## When to use

Use when running the SpaceM pipeline on MALDI-imaging data (*.RAW, *.UDP, *.imzML, *.ibd) plus pre/post-MALDI tiled microscopy, and per-cell segmentation via a CellProfiler pipeline must be configured in paths.json.

## When NOT to use

- Data is not SpaceM-format tiled microscopy paired with MALDI-imaging files — the CP pipeline/project paths are consumed specifically by runAnalysis.py
- Images are already single pre-stitched fields without tiles and out.txt coordinates — the Nikon Ti-E tile-stitching workflow does not apply

## Inputs

- Post-MALDI tiled microscopy tile images (*.tif) with out.txt tile coordinate files (Nikon Ti-E format)
- Pre-MALDI tiled microscopy images with penmarks
- CellProfiler pipeline file and CellProfiler project file capable of segmenting the cultured cells
- paths.json configuration file

## Outputs

- Per-cell segmentation masks used by SpaceM during runAnalysis.py execution
- Cell-level entries in the final spatio-molecular matrix MORPHnMOL.csv (stored in MF/Analysis/scAnalysis)

## How to apply

Download and install CellProfiler 3.0.0 (per the README installation section) and supply CP pipeline and CP project files that are able to segment the cultured cells from the tiled microscopy images. In paths.json, add the CellProfiler executable path under 'CellProfiler path', the pipeline file under 'CellProfiler pipeline path', and the project file under 'CellProfiler project path', alongside the MF main folder and Fiji paths. Segmentation then runs automatically when 'python runAnalysis.py' is executed: tiles (optimized for Nikon Ti-E output format with out.txt upper-left pixel coordinates) are stitched, cells are segmented with the CP pipeline, and ablation marks are co-registered with cells so that locally ablated cell parts are identified. The rationale is that precise cell segmentation is the cornerstone step enabling metabolite intensities to be attributed to individual cells rather than pixels.

## Related tools

- **CellProfiler** (Provides the segmentation software; the CP pipeline and CP project files that segment cells are registered in paths.json and invoked by SpaceM during runAnalysis.py) — https://cellprofiler.org/previous_releases/
- **SpaceM** (Main pipeline that stitches tiles, applies the CellProfiler segmentation, and produces the spatio-molecular matrix MORPHnMOL.csv) — https://github.com/alexandrovteam/SpaceM
- **Python** (Execution environment; runAnalysis.py prompts the user through stitching and CellProfiler segmentation steps)

## Examples

```
python runAnalysis.py
```

## Evaluation signals

- paths.json contains non-empty values for 'CellProfiler path', 'CellProfiler pipeline path', and 'CellProfiler project path' alongside the MF and Fiji entries
- runAnalysis.py completes without prompting errors and stitches the Nikon Ti-E tiles successfully
- MORPHnMOL.csv exists in MF/Analysis/scAnalysis after execution, confirming segmentation fed into the spatio-molecular matrix
- Segmented cell count is consistent with cell confluency in the microscopy tiles, and cells align with non-overlapping visible ablation marks in post-MALDI microscopy

## Limitations

- No version numbers are given in the article methods for CellProfiler; the README specifies CellProfiler 3.0.0 and a Fiji release of December 22 2015
- Tile stitching is optimized only for the Nikon Ti-E microscope output format, constraining which raw microscopy exports can be segmented without conversion
- Segmentation quality depends on ablation marks being visible and non-overlapping in the post-MALDI microscopy; overlapping marks cannot be cleanly assigned

## Evidence

- [methods] The CellProfiler (CP) software, CP pipeline for segmentation, and CP project files able to segment the cells are required.: "The CellProfiler (CP) software, CP pipeline for segmentation, and CP project files able to segment the cells are required."
- [methods] Run `python runAnalysis.py` and follow instructions when prompted.: "Run `python runAnalysis.py` and follow instructions when prompted."
- [readme] At the moment, SpaceM requires the stitching of the tiles and this particular code is optimized for the Nikon Ti-E microscope output format.: "At the moment, SpaceM requires the stitching of the tiles and this particular code is optimized for the Nikon Ti-E microscope output format."
- [readme] Download and install CellProfiler 3.0.0: "Download and install [CellProfiler 3.0.0](https://cellprofiler.org/previous_releases/)"
- [readme] It is critical that ablation marks are visible and non-overlapping in the post-MALDI microscopy.: "It is critical that ablation marks are visible and non-overlapping in the post-MALDI microscopy."
