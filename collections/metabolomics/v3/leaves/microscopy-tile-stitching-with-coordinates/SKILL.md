---
name: microscopy-tile-stitching-with-coordinates
description: Use when SpaceM input contains tiled microscopy acquisitions (tile_1.tif,
  tile_2.tif, ...) each paired with an out.txt coordinate file, and tiles must be
  merged before cell segmentation and MALDI-ablation co-registration.
when_to_use_negative:
- Images are already pre-stitched — the README notes the next implementation version
  will accept pre-stitched images, but the current pipeline expects tiles plus out.txt
  coordinates
- Tile coordinate format does not match Nikon Ti-E microscope output — this code is
  explicitly optimized only for that format
- Post-MALDI microscopy lacks visible or has overlapping ablation marks — co-registration
  of metabolite intensities to cells will be unreliable
edam_operation: http://edamontology.org/operation_3502
edam_topics:
- http://edamontology.org/topic_3382
- http://edamontology.org/topic_3172
tools:
- name: SpaceM
  role: Pipeline that performs the stitching via runAnalysis.py and downstream cell
    segmentation/co-registration
  repo: github.com/alexandrovteam/SpaceM
- name: CellProfiler
  role: Segments cells on stitched microscopy images; its path and pipeline/project
    paths are registered in paths.json
  repo: https://cellprofiler.org/previous_releases/
- name: Fiji
  role: Image-analysis dependency whose executable path must be registered in paths.json
    before running the pipeline
  repo: https://imagej.net/Fiji/Downloads
- name: Nikon Ti-E
  role: Microscope whose tiled-acquisition output format the stitching code is optimized
    for
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41592-021-01198-0
    title: SpaceM reveals metabolic states of single cells
  source_content_hash: sha256:f12e2d4d8c24a5219c713a8028dc8e74328e68ef7e81826555226e54cdd14dab
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - CellProfiler
  - Fiji
  - Nikon Ti-E
  - Python
  - SpaceM
  edam_operation: http://edamontology.org/operation_3502
  edam_topics:
  - http://edamontology.org/topic_3382
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4166666666666667
n_claims: 12
---

# microscopy-tile-stitching-with-coordinates

## Summary

Stitch tiled pre- and post-MALDI microscopy images using per-tile upper-left pixel x-y coordinates from out.txt files, enabling SpaceM's subcellular co-registration of cells with MALDI laser ablation marks.

## When to use

Use when SpaceM input contains tiled microscopy acquisitions (tile_1.tif, tile_2.tif, ...) each paired with an out.txt coordinate file, and tiles must be merged before cell segmentation and MALDI-ablation co-registration.

## When NOT to use

- Images are already pre-stitched — the README notes the next implementation version will accept pre-stitched images, but the current pipeline expects tiles plus out.txt coordinates
- Tile coordinate format does not match Nikon Ti-E microscope output — this code is explicitly optimized only for that format
- Post-MALDI microscopy lacks visible or has overlapping ablation marks — co-registration of metabolite intensities to cells will be unreliable

## Inputs

- preMALDI tile images (*.tif) with out.txt upper-left pixel x-y coordinate file (um)
- postMALDI tile images (*.tif) with out.txt upper-left pixel x-y coordinate file (um), ablation marks visible and non-overlapping
- paths.json registering MF path, CellProfiler path, Fiji path, and CP pipeline/project paths
- MALDI imaging data (*.RAW, *.UDP, *.imzML, *.ibd) in MF/Input/MALDI

## Outputs

- Stitched pre- and post-MALDI microscopy images used for cell segmentation and ablation-mark co-registration
- Final spatio-molecular matrix MORPHnMOL.csv in MF/Analysis/scAnalysis

## How to apply

Organize preMALDI and postMALDI tile TIFFs under MF/Input/Microscopy, each subfolder containing an out.txt with the upper-left pixel x-y coordinates in micrometers of every tile. Run 'python runAnalysis.py' and follow the prompts so SpaceM stitches the tiles; the implementation is optimized for the Nikon Ti-E microscope output format. Post-MALDI microscopy must show ablation marks that are visible and non-overlapping, since these marks are what get co-registered with segmented cells at subcellular precision to assign metabolite intensities to individual cells. Register MF, CellProfiler, Fiji, and CellProfiler pipeline/project paths in paths.json before execution.

## Related tools

- **SpaceM** (Pipeline that performs the stitching via runAnalysis.py and downstream cell segmentation/co-registration) — github.com/alexandrovteam/SpaceM
- **CellProfiler** (Segments cells on stitched microscopy images; its path and pipeline/project paths are registered in paths.json) — https://cellprofiler.org/previous_releases/
- **Fiji** (Image-analysis dependency whose executable path must be registered in paths.json before running the pipeline) — https://imagej.net/Fiji/Downloads
- **Nikon Ti-E** (Microscope whose tiled-acquisition output format the stitching code is optimized for)

## Examples

```
python runAnalysis.py
```

## Evaluation signals

- MORPHnMOL.csv exists in MF/Analysis/scAnalysis after the run
- runAnalysis.py completed without prompting errors
- Both preMALDI and postMALDI subfolders contain tile TIFFs plus an out.txt with upper-left pixel x-y coordinates in um
- Ablation marks are visible and non-overlapping in the stitched post-MALDI microscopy
- Tiles were stitched successfully in the Nikon Ti-E output format without coordinate-mismatch errors

## Limitations

- Stitching code is optimized for the Nikon Ti-E microscope output format; other formats are a limitation of this implementation, not of the SpaceM method
- Pre-stitched images are not accepted by the current implementation
- It is critical that ablation marks are visible and non-overlapping in the post-MALDI microscopy, otherwise the workflow fails
- No version numbers are given for CellProfiler, Fiji, or Python beyond CellProfiler 3.0.0 and the Fiji release of December 22, 2015

## Evidence
