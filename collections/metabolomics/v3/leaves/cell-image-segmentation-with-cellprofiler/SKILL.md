---
name: cell-image-segmentation-with-cellprofiler
description: Use when running the SpaceM workflow on MALDI-imaging mass spectrometry
  data from cultured cells and you need per-cell segmentation of the stitched pre-MALDI
  microscopy tiles to assign metabolite intensities to individual cells.
when_to_use_negative:
- Input is a single-cell gene-expression dataset or already-segmented cell masks —
  CellProfiler segmentation of pre-MALDI tiles is specific to SpaceM's microscopy
  + MALDI-IMS integration.
- The microscopy is not a tiled acquisition with black penmarks and out.txt tile coordinates;
  SpaceM currently requires tile stitching optimized for the Nikon Ti-E microscope
  output format.
- Ablation marks are overlapping or not visible in the post-MALDI microscopy, which
  the README states is critical for correct co-registration.
edam_operation: http://edamontology.org/operation_3443
edam_topics:
- http://edamontology.org/topic_3382
- http://edamontology.org/topic_3391
tools:
- name: CellProfiler
  role: Executes the CP pipeline and project files that segment the cells in the stitched
    pre-MALDI microscopy image
  repo: https://cellprofiler.org/previous_releases/
- name: Fiji
  role: Used by SpaceM for stitching the pre-/post-MALDI microscopy tiles prior to
    segmentation; its path must be registered in paths.json
  repo: https://imagej.net/Fiji/Downloads
- name: SpaceM
  role: Orchestrates the segmentation step and co-registers segmented cells with MALDI
    ablation marks
  repo: github.com/alexandrovteam/SpaceM
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41592-021-01198-0
    title: SpaceM reveals metabolic states of single cells
  source_content_hash: sha256:6293f29f79a88ff9b07f8d28edd7e463c605b3715c6b9250d9991fcf34fb2098
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - CellProfiler
  - Fiji
  - Nikon Ti-E
  - Python
  - SpaceM
  edam_operation: http://edamontology.org/operation_3443
  edam_topics:
  - http://edamontology.org/topic_3382
  - http://edamontology.org/topic_3391
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# cell-image-segmentation-with-cellprofiler

## Summary

Segments individual cultured cells in the stitched pre-MALDI microscopy image using a CellProfiler pipeline, producing per-cell segmentation masks that SpaceM co-registers with MALDI laser ablation marks to build the per-cell spatio-molecular matrix.

## When to use

Use when running the SpaceM workflow on MALDI-imaging mass spectrometry data from cultured cells and you need per-cell segmentation of the stitched pre-MALDI microscopy tiles to assign metabolite intensities to individual cells.

## When NOT to use

- Input is a single-cell gene-expression dataset or already-segmented cell masks — CellProfiler segmentation of pre-MALDI tiles is specific to SpaceM's microscopy + MALDI-IMS integration.
- The microscopy is not a tiled acquisition with black penmarks and out.txt tile coordinates; SpaceM currently requires tile stitching optimized for the Nikon Ti-E microscope output format.
- Ablation marks are overlapping or not visible in the post-MALDI microscopy, which the README states is critical for correct co-registration.

## Inputs

- Stitched pre-MALDI microscopy image of the cell culture area (from tiled .tif acquisitions with out.txt upper-left pixel x-y coordinates in um)
- CellProfiler pipeline (.cppipe) and CellProfiler project files able to segment the cells
- paths.json with entries 'CellProfiler path', 'CellProfiler pipeline path', 'CellProfiler project path', 'Fiji path', and 'MF'

## Outputs

- Per-cell segmentation masks of cells in the pre-MALDI microscopy image
- Cell segmentation features joined into the final spatio-molecular matrix MORPHnMOL.csv (in MF/Analysis/scAnalysis), containing normalized metabolic profiles plus microscopy-derived phenotypic properties per segmented cell

## How to apply

Create the main folder `MF` with the Input/Microscopy/preMALDI tiled acquisitions (tile_*.tif plus out.txt tile coordinates in um) and stitch the tiles using the Nikon Ti-E-optimized stitching code. Configure CellProfiler with a CP pipeline and CP project files able to segment the cells in the stitched pre-MALDI image, and register the CellProfiler executable, pipeline path, and project path in `paths.json` alongside the Fiji path and MF path. Run SpaceM via `python runAnalysis.py` and follow the interactive prompts, which invoke the CellProfiler segmentation as part of the co-registration of ablation marks (segmented from post-MALDI microscopy) with the segmented cells, applying the 45% sampling-area overlap normalization. The rationale is that SpaceM's cornerstone is the precise estimation of which cell parts were locally ablated by the laser, which requires accurate cell masks from the pre-MALDI image.

## Related tools

- **CellProfiler** (Executes the CP pipeline and project files that segment the cells in the stitched pre-MALDI microscopy image) — https://cellprofiler.org/previous_releases/
- **Fiji** (Used by SpaceM for stitching the pre-/post-MALDI microscopy tiles prior to segmentation; its path must be registered in paths.json) — https://imagej.net/Fiji/Downloads
- **SpaceM** (Orchestrates the segmentation step and co-registers segmented cells with MALDI ablation marks) — github.com/alexandrovteam/SpaceM

## Examples

```
python runAnalysis.py
```

## Evaluation signals

- Segmented cell masks accurately outline cells in the stitched pre-MALDI image (configured via CellProfiler project files able to segment the cells).
- Co-registration of ablation marks with segmented cells succeeds, with ablation marks visible and non-overlapping in the post-MALDI microscopy.
- The SpaceM run completes and the final spatio-molecular matrix MORPHnMOL.csv is written inside the scAnalysis sub-folder of MF/Analysis.
- MORPHnMOL.csv contains normalized metabolic profiles joined to microscopy-derived phenotypic properties (e.g., cell fluorescence intensities and morpho-spatial features) for each segmented cell.
- CellProfiler version 3.0.0 and the specified Fiji release (December 22, 2015) are installed, matching the README requirements.

## Limitations

- SpaceM requires the stitching of the tiles and the stitching code is optimized for the Nikon Ti-E microscope output format; pre-stitched images are not accepted in this implementation.
- No version numbers are given in the methods beyond the README's CellProfiler 3.0.0 and Fiji December 22 2015 release; other versions are untested in this context.
- Ablation marks must be visible and non-overlapping in the post-MALDI microscopy, otherwise per-cell assignment fails.

## Evidence

- [methods] CellProfiler pipeline and project files required for segmentation: "The CellProfiler (CP) software, CP pipeline for segmentation, and CP project files able to segment the cells are required."
- [intro] Co-registration is the cornerstone of SpaceM: "The cornerstone of SpaceM is the precise estimation of which cell parts were locally ablated by the laser, achieved by co-registering microscopy images of cells with microscopy images of MALDI laser"
- [readme] Tile acquisition and stitching requirements: "For both pre- and post-MALDI images, a tiled acquisition of the cell culture area with black penmarks is required."
- [readme] Output spatio-molecular matrix location: "The final spatio-molecular matrix will be stored as `MORPHnMOL.csv` and can be found inside the `scAnalysis` sub-folder."
