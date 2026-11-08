---
name: maldi-ablation-mark-detection-in-post-maldi-microscopy
description: Use when processing MALDI-imaging mass spectrometry data of cultured
  cells where per-cell metabolite assignment requires detecting laser ablation marks
  in post-MALDI microscopy and linking them to segmented cells.
when_to_use_negative:
- Ablation marks are overlapping or not visible in the post-MALDI microscopy (README
  states this is critical and invalidates the registration)
- Aim is single-cell metabolomics of tissue sections or non-adhered cells rather than
  cultured cells acquired as tiled microscopy with fiducial pen marks
- You already have pre-stitched microscopy images — the current SpaceM implementation
  requires raw tiles and out.txt coordinates (a limitation of the implementation,
  not the method)
edam_operation: http://edamontology.org/operation_3634
edam_topics:
- http://edamontology.org/topic_3382
- http://edamontology.org/topic_0121
tools:
- name: SpaceM
  role: Implements ablation-mark detection, co-registration of pre/post-MALDI microscopy,
    and generation of the spatio-molecular matrix
  repo: github.com/alexandrovteam/SpaceM
- name: CellProfiler
  role: Segmentation of cells in the stitched pre-MALDI microscopy image via CP pipeline
    and project files
- name: Fiji
  role: Image processing backend whose path must be registered in paths.json for the
    SpaceM analysis
- name: METASPACE
  role: Provides per-mark molecular profiles from MALDI-imaging data that are joined
    to the co-registered ablation marks
- name: Nikon Ti-E
  role: Microscope whose tile output format the stitching code is optimized for
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41592-021-01198-0
    title: SpaceM reveals metabolic states of single cells
  source_content_hash: sha256:2547787748726ebbb354285fc5c61541305df420b568a68b1c7c60808b659c70
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
  edam_operation: http://edamontology.org/operation_3634
  edam_topics:
  - http://edamontology.org/topic_3382
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7058823529411765
n_claims: 17
---

# maldi-ablation-mark-detection-in-post-maldi-microscopy

## Summary

Detect MALDI laser ablation marks in post-MALDI microscopy tiles and co-register them with pre-MALDI cell segmentations at subcellular precision, so that per-mark molecular profiles from METASPACE can be assigned to individual cells in the SpaceM pipeline.

## When to use

Use when processing MALDI-imaging mass spectrometry data of cultured cells where per-cell metabolite assignment requires detecting laser ablation marks in post-MALDI microscopy and linking them to segmented cells.

## When NOT to use

- Ablation marks are overlapping or not visible in the post-MALDI microscopy (README states this is critical and invalidates the registration)
- Aim is single-cell metabolomics of tissue sections or non-adhered cells rather than cultured cells acquired as tiled microscopy with fiducial pen marks
- You already have pre-stitched microscopy images — the current SpaceM implementation requires raw tiles and out.txt coordinates (a limitation of the implementation, not the method)

## Inputs

- post-MALDI microscopy tile images (tile_1.tif, ...) with out.txt tile coordinates in um
- pre-MALDI microscopy tile images of the same cell culture area
- MALDI-imaging files: .RAW, .UDP, .imzML, .ibd
- CellProfiler pipeline and project files able to segment the cells
- paths.json with MF, CellProfiler pipeline/project, and Fiji paths
- Black pen-mark fiducials in the tiled microscopy acquisitions

## Outputs

- Co-registration of ablation marks to segmented cells (per-mark-to-cell assignment)
- Per-cell spatio-molecular matrix MORPHnMOL.csv in MF/Analysis/scAnalysis

## How to apply

Organize the MF folder with Input/MALDI (.RAW, .UDP, .imzML, .ibd) and Input/Microscopy subfolders (preMALDI and postMALDI tiles plus out.txt with upper-left pixel x-y coordinates in um), register MF, CellProfiler, and Fiji paths in paths.json. Stitch the pre- and post-MALDI tiles using the out.txt coordinates with the Nikon Ti-E-optimized stitching code. Segment cells in the stitched pre-MALDI image with the CellProfiler pipeline/project, then detect the MALDI laser ablation marks in the post-MALDI image, using the black pen-mark fiducials to co-register the two images and determine which cell parts were locally ablated. Apply the 45% sampling-area overlap normalization when assigning ablated pixels to cells, and require that ablation marks are visible and non-overlapping in the post-MALDI microscopy. The downstream join with METASPACE per-mark molecular profiles produces the spatio-molecular matrix MORPHnMOL.csv.

## Related tools

- **SpaceM** (Implements ablation-mark detection, co-registration of pre/post-MALDI microscopy, and generation of the spatio-molecular matrix) — github.com/alexandrovteam/SpaceM
- **CellProfiler** (Segmentation of cells in the stitched pre-MALDI microscopy image via CP pipeline and project files)
- **Fiji** (Image processing backend whose path must be registered in paths.json for the SpaceM analysis)
- **METASPACE** (Provides per-mark molecular profiles from MALDI-imaging data that are joined to the co-registered ablation marks)
- **Nikon Ti-E** (Microscope whose tile output format the stitching code is optimized for)

## Examples

```
python runAnalysis.py
```

## Evaluation signals

- Pipeline runs to completion via 'python runAnalysis.py' with interactive prompts and writes MORPHnMOL.csv inside the scAnalysis sub-folder of MF/Analysis
- Ablation marks are visible and non-overlapping in the post-MALDI microscopy before registration
- Co-registration aligns pre- and post-MALDI images via black pen-mark fiducials such that ablated cell parts are correctly identified per cell
- Per-cell metabolic intensities reflect the 45% sampling-area overlap normalization
- MORPHnMOL.csv joins normalized metabolic profiles to microscopy-derived phenotypic properties (fluorescence intensities, morpho-spatial features) for each segmented cell

## Limitations

- Tile stitching code is optimized for the Nikon Ti-E microscope output format; other microscopes' out.txt formats are not directly supported (implementation limitation, not a method limitation)
- Pre-stitched images are not accepted by the current implementation
- Co-sampling of neighboring cells by the same ablation mark can occur, though the authors report it minimally affects cell-type prediction accuracy and did not change metabolic state markers

## Evidence

- [full_text] SpaceM identifies the sampled cell areas by co-registering the pre-MALDI microscopy image with a second round of microscopy images, called post-MALDI, from which the MALDI laser ablation marks are segmented.: [evidence span withheld — anchor status: not_found]
