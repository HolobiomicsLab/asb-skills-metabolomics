---
name: fiducial-based-image-coregistration-with-penmarks
description: Use when pre- and post-MALDI tiled microscopy images must be stitched
  and aligned so MALDI ablation marks can be assigned to segmented cells in a SpaceM-style
  workflow.
when_to_use_negative:
- The microscopy tiles are already pre-stitched and no tile co-registration is needed
  (the current implementation requires stitching of tiles; pre-stitched input support
  is planned for a future version)
- Ablation marks are overlapping or not visible in the post-MALDI microscopy — the
  co-registration of marks to cells cannot be done reliably
- The data are not cultured cells imaged with MALDI-imaging mass spectrometry plus
  light microscopy, e.g. single-cell transcriptomics or bulk metabolomics data
edam_operation: http://edamontology.org/operation_3443
edam_topics:
- http://edamontology.org/topic_3382
- http://edamontology.org/topic_0092
tools:
- name: SpaceM
  role: Executes the stitching and penmark-based co-registration of pre-/post-MALDI
    microscopy with ablation mark segmentation (run via runAnalysis.py)
  repo: github.com/alexandrovteam/SpaceM
- name: Fiji
  role: Image analysis backend registered via 'Fiji path' in paths.json, used for
    stitching the microscopy tiles
- name: CellProfiler
  role: Segments cells in the stitched pre-MALDI image using the CP pipeline and CP
    project registered in paths.json
- name: Nikon Ti-E
  role: Microscope whose tile output format (out.txt coordinates) the stitching code
    is optimized for
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41592-021-01198-0
    title: SpaceM reveals metabolic states of single cells
  source_content_hash: sha256:6ebc7b8b074b04144db086c9a128ebd43785bc29711761ebd99536b6b4097100
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
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7857142857142857
n_claims: 14
---

# Fiducial-based image co-registration with penmarks

## Summary

Co-registers pre- and post-MALDI microscopy tiles of a cell culture, using black penmark fiducials and out.txt tile coordinates, to spatially align MALDI laser ablation marks to segmented cells at subcellular precision for per-cell metabolite assignment.

## When to use

Use when pre- and post-MALDI tiled microscopy images must be stitched and aligned so MALDI ablation marks can be assigned to segmented cells in a SpaceM-style workflow.

## When NOT to use

- The microscopy tiles are already pre-stitched and no tile co-registration is needed (the current implementation requires stitching of tiles; pre-stitched input support is planned for a future version)
- Ablation marks are overlapping or not visible in the post-MALDI microscopy — the co-registration of marks to cells cannot be done reliably
- The data are not cultured cells imaged with MALDI-imaging mass spectrometry plus light microscopy, e.g. single-cell transcriptomics or bulk metabolomics data

## Inputs

- Pre-MALDI tiled microscopy images (tile_1.tif, tile_2.tif, ...) with out.txt upper-left pixel x-y coordinates in um
- Post-MALDI tiled microscopy images (tile_1.tif, tile_2.tif, ...) showing ablation marks, with out.txt coordinates
- Black penmark fiducials visible in both image sets
- CellProfiler segmentation of cells from the stitched pre-MALDI image (CP pipeline and CP project registered in paths.json)

## Outputs

- Co-registered mapping of each segmented MALDI ablation mark to the underlying cell regions, feeding the per-cell spatio-molecular matrix (MORPHnMOL.csv)

## How to apply

Acquire tiled pre-MALDI and post-MALDI microscopy of the cell culture with black penmarks as fiducials, storing tiles with an out.txt text file containing the upper-left pixel x-y coordinates (in um) of each tile. Stitch both tile sets using the Nikon Ti-E-optimized stitching code (Fiji is registered via the 'Fiji path' in paths.json), then co-register the stitched pre- and post-MALDI images via the penmark fiducials. In the post-MALDI image, segment the MALDI laser ablation marks — these must be visible and non-overlapping — and map each mark onto the CellProfiler-segmented cells from the pre-MALDI image to determine which cell parts were locally ablated, applying the 45% sampling-area overlap normalization when a mark partially samples a cell. The rationale is that precise estimation of locally ablated cell parts is the cornerstone of assigning MALDI metabolite intensities to individual cells.

## Related tools

- **SpaceM** (Executes the stitching and penmark-based co-registration of pre-/post-MALDI microscopy with ablation mark segmentation (run via runAnalysis.py)) — github.com/alexandrovteam/SpaceM
- **Fiji** (Image analysis backend registered via 'Fiji path' in paths.json, used for stitching the microscopy tiles)
- **CellProfiler** (Segments cells in the stitched pre-MALDI image using the CP pipeline and CP project registered in paths.json)
- **Nikon Ti-E** (Microscope whose tile output format (out.txt coordinates) the stitching code is optimized for)

## Examples

```
python runAnalysis.py
```

## Evaluation signals

- Pipeline completes and the spatio-molecular matrix MORPHnMOL.csv is written inside the scAnalysis sub-folder of MF/Analysis
- In the post-MALDI stitched image, all ablation marks are visible and non-overlapping
- Penmark fiducials are present in both pre- and post-MALDI stitched images and the co-registration aligns them
- The 45% sampling-area overlap normalization is applied for ablation marks that partially sample cells
- Downstream per-cell metabolic profiles join METASPACE per-mark molecular profiles to the correct co-registered marks

## Limitations

- The stitching code is optimized for the Nikon Ti-E microscope output format only; other formats may not work in this implementation, though this is an implementation limitation and not a limitation of the SpaceM method
- Pre-stitched images are not accepted by the current implementation; tiled acquisition with out.txt coordinates is required
- Co-sampling of neighboring cells by the same ablation mark occurs, but was shown to only minimally affect cell-type prediction accuracy in spatially heterogeneous co-cultures

## Evidence
