---
name: microscopy-tile-stitching-from-coordinate-files
description: Use when input microscopy consists of multiple tiles (e.g., tile_1.tif,
  tile_2.tif) accompanied by an out.txt coordinate file, and the tiles must be reassembled
  into a stitched pre- or post-MALDI image for ablation-mark-to-cell co-registration.
when_to_use_negative:
- The microscopy images are already pre-stitched — the README notes a future implementation
  'will also be accepting pre-stitched images', so this tile-stitching step is unnecessary
  then
- The microscope output format is not Nikon Ti-E — the stitching code is explicitly
  optimized for that format and may not parse other coordinate files
- Ablation marks are overlapping or not visible in the post-MALDI microscopy — stitching
  alone cannot rescue the co-registration requirement
edam_operation: http://edamontology.org/operation_3552
edam_topics:
- http://edamontology.org/topic_3382
- http://edamontology.org/topic_3385
tools:
- name: Fiji
  role: Image-analysis tool whose path must be registered in paths.json for the SpaceM
    stitching/processing pipeline
  repo: github.com/alexandrovteam/SpaceM
- name: SpaceM
  role: Pipeline that invokes the Nikon Ti-E-optimized tile-stitching code as a prerequisite
    for ablation-mark co-registration
  repo: github.com/alexandrovteam/SpaceM
- name: Nikon Ti-E
  role: Microscope whose tile output format (out.txt upper-left pixel x-y coordinates
    in um) the stitching code is optimized for
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41592-021-01198-0
    title: SpaceM reveals metabolic states of single cells
  source_content_hash: sha256:20842a2463ed87564d2448acfc03093a787dc62a24dcf3ed8740e143cb0e8f5d
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Fiji
  - Nikon Ti-E
  - Python
  - SpaceM
  edam_operation: http://edamontology.org/operation_3552
  edam_topics:
  - http://edamontology.org/topic_3382
  - http://edamontology.org/topic_3385
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# microscopy-tile-stitching-from-coordinate-files

## Summary

Stitches tiled pre- and post-MALDI microscopy acquisitions of a cell culture into whole-field images using the upper-left pixel x-y coordinates (in um) recorded in each tile's out.txt file, enabling co-registration of cells with MALDI laser ablation marks in SpaceM.

## When to use

Use when input microscopy consists of multiple tiles (e.g., tile_1.tif, tile_2.tif) accompanied by an out.txt coordinate file, and the tiles must be reassembled into a stitched pre- or post-MALDI image for ablation-mark-to-cell co-registration.

## When NOT to use

- The microscopy images are already pre-stitched — the README notes a future implementation 'will also be accepting pre-stitched images', so this tile-stitching step is unnecessary then
- The microscope output format is not Nikon Ti-E — the stitching code is explicitly optimized for that format and may not parse other coordinate files
- Ablation marks are overlapping or not visible in the post-MALDI microscopy — stitching alone cannot rescue the co-registration requirement

## Inputs

- Pre-MALDI microscopy tiles (tile_1.tif, tile_2.tif, ...) with out.txt coordinate file
- Post-MALDI microscopy tiles (tile_1.tif, tile_2.tif, ...) with out.txt coordinate file
- out.txt text file containing the upper-left pixel x-y coordinates in um of each tile

## Outputs

- Stitched pre-MALDI microscopy image for cell segmentation
- Stitched post-MALDI microscopy image with detectable, non-overlapping laser ablation marks for co-registration

## How to apply

Acquire tiled images of the cell culture area with black pen-mark fiducials for both pre-MALDI and post-MALDI rounds; each tile must be stored with an out.txt file containing the upper-left pixel x-y coordinates in um. Apply the stitching code optimized for the Nikon Ti-E microscope output format to merge the tiles into full-field pre- and post-MALDI images. The stitched post-MALDI image is then used to segment the MALDI laser ablation marks, which must be visible and non-overlapping; the stitched pre-MALDI image feeds CellProfiler cell segmentation. Stitching is a prerequisite step before registering paths (MF, CellProfiler pipeline/project, Fiji) in paths.json and running runAnalysis.py.

## Related tools

- **Fiji** (Image-analysis tool whose path must be registered in paths.json for the SpaceM stitching/processing pipeline) — github.com/alexandrovteam/SpaceM
- **SpaceM** (Pipeline that invokes the Nikon Ti-E-optimized tile-stitching code as a prerequisite for ablation-mark co-registration) — github.com/alexandrovteam/SpaceM
- **Nikon Ti-E** (Microscope whose tile output format (out.txt upper-left pixel x-y coordinates in um) the stitching code is optimized for)

## Examples

```
python runAnalysis.py
```

## Evaluation signals

- Stitched pre- and post-MALDI images form a complete, gap-free field of the cell culture area with pen-mark fiducials aligned across the two rounds
- Ablation marks are visible and non-overlapping in the stitched post-MALDI image — stated as critical in the README
- Tile positions in the stitched image match the out.txt upper-left pixel x-y coordinates in um for every tile
- Downstream SpaceM run completes and writes MORPHnMOL.csv inside the scAnalysis sub-folder of MF/Analysis

## Limitations

- The stitching code is optimized only for the Nikon Ti-E microscope output format; other formats are not supported in this implementation
- Pre-stitched images are not accepted by this version of the implementation
- It is critical that ablation marks are visible and non-overlapping in the post-MALDI microscopy; failure here breaks the mark-to-cell assignment

## Evidence
