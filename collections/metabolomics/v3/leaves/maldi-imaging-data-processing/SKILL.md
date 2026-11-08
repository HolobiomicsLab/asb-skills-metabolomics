---
name: maldi-imaging-data-processing
description: Use when raw MALDI-imaging data (*.RAW, *.UDP, *.imzML, *.ibd) plus tiled
  pre-/post-MALDI microscopy tiles with out.txt coordinates must be converted into
  a per-cell spatio-molecular matrix via the SpaceM pipeline.
when_to_use_negative:
- Microscopy is already pre-stitched rather than supplied as tiled acquisitions with
  out.txt coordinates (current code requires tile stitching and is optimized for Nikon
  Ti-E output format)
- Post-MALDI microscopy lacks visible or contains overlapping ablation marks, so per-cell
  metabolite intensity assignment cannot be trusted
- Data are not MALDI-imaging mass spectrometry of cultured cells paired with light
  microscopy (e.g., bulk LC-MS/MS tissue measurements or scRNA-seq)
tools:
- name: SpaceM
  role: Open-source pipeline integrating MALDI-imaging mass spectrometry and light
    microscopy; driven via runAnalysis.py to produce MORPHnMOL.csv
  repo: github.com/alexandrovteam/SpaceM
- name: CellProfiler
  role: Runs the CP segmentation pipeline/project that segments cells from microscopy
    images; path registered as 'CellProfiler path' in paths.json
- name: Fiji
  role: Image-processing dependency used for tile registration/preprocessing; path
    registered as 'Fiji path' in paths.json
- name: METASPACE
  role: Web platform for MALDI-imaging metabolite annotation/analysis of the .RAW/.UDP/.imzML/.ibd
    data
  repo: https://metaspace2020.eu/
- name: Python
  role: Runtime for runAnalysis.py with python3 and dependencies installed via requirements.txt
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41592-021-01198-0
    title: SpaceM reveals metabolic states of single cells
  source_content_hash: sha256:6b9e04c08371699c010a15a8dde49896ad17f7675fd06113fb592a7d3be0731b
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - CellProfiler
  - Fiji
  - METASPACE
  - Python
  - SpaceM
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 12
---

# maldi-imaging-data-processing

## Summary

Runs the SpaceM single-cell metabolomics pipeline on paired MALDI-imaging mass spectrometry files (.RAW, .UDP, .imzML, .ibd) and tiled pre-/post-MALDI microscopy, producing a per-cell spatio-molecular matrix (MORPHnMOL.csv) of normalized metabolite intensities plus microscopy-derived morphometric features.

## When to use

Use when raw MALDI-imaging data (*.RAW, *.UDP, *.imzML, *.ibd) plus tiled pre-/post-MALDI microscopy tiles with out.txt coordinates must be converted into a per-cell spatio-molecular matrix via the SpaceM pipeline.

## When NOT to use

- Microscopy is already pre-stitched rather than supplied as tiled acquisitions with out.txt coordinates (current code requires tile stitching and is optimized for Nikon Ti-E output format)
- Post-MALDI microscopy lacks visible or contains overlapping ablation marks, so per-cell metabolite intensity assignment cannot be trusted
- Data are not MALDI-imaging mass spectrometry of cultured cells paired with light microscopy (e.g., bulk LC-MS/MS tissue measurements or scRNA-seq)

## Inputs

- MALDI-imaging files: *.RAW, *.UDP, *.imzML, *.ibd in Input/MALDI
- Tiled pre-MALDI microscopy images (tile_1.tif, tile_2.tif, ...) with out.txt tile-coordinate file
- Tiled post-MALDI microscopy images with out.txt tile-coordinate file, with visible, non-overlapping ablation marks
- CellProfiler segmentation pipeline (.pipeline) and CellProfiler project files
- paths.json configuration file

## Outputs

- Final spatio-molecular matrix MORPHnMOL.csv in MF/Analysis/scAnalysis (normalized per-cell metabolic profiles plus microscopy-derived phenotypic properties such as fluorescence intensities and morpho-spatial features)
- Intermediate results in the MF/Analysis sub-folder

## How to apply

Manually organize a main folder MF with Input/MALDI (.RAW, .UDP, .imzML, .ibd) and Input/Microscopy subfolders holding preMALDI and postMALDI tile_*.tif images each accompanied by an out.txt of upper-left pixel x-y coordinates in um. Analyze the MALDI data with METASPACE (https://metaspace2020.eu/), first confirming ablation marks are visible and non-overlapping in the post-MALDI microscopy. Register paths in paths.json: the MF path, CellProfiler path (with the CP segmentation pipeline and project paths), and Fiji path. Run 'python runAnalysis.py' following prompted instructions so that tiles are stitched (code is optimized for Nikon Ti-E output format, requiring unstitched tiles) and cells are segmented with the CellProfiler pipeline; results land in the Analysis sub-folder and the final matrix MORPHnMOL.csv lands in the scAnalysis sub-folder.

## Related tools

- **SpaceM** (Open-source pipeline integrating MALDI-imaging mass spectrometry and light microscopy; driven via runAnalysis.py to produce MORPHnMOL.csv) — github.com/alexandrovteam/SpaceM
- **CellProfiler** (Runs the CP segmentation pipeline/project that segments cells from microscopy images; path registered as 'CellProfiler path' in paths.json)
- **Fiji** (Image-processing dependency used for tile registration/preprocessing; path registered as 'Fiji path' in paths.json)
- **METASPACE** (Web platform for MALDI-imaging metabolite annotation/analysis of the .RAW/.UDP/.imzML/.ibd data) — https://metaspace2020.eu/
- **Python** (Runtime for runAnalysis.py with python3 and dependencies installed via requirements.txt)

## Examples

```
python runAnalysis.py
```

## Evaluation signals

- MORPHnMOL.csv exists inside MF/Analysis/scAnalysis, matching the README's stated output location
- runAnalysis.py executed to completion without prompting errors after paths.json registration
- MF folder structure matches the README schema (Input/MALDI files and Input/Microscopy/preMALDI and postMALDI tiles each with out.txt)
- Ablation marks are verified visible and non-overlapping in the post-MALDI microscopy before METASPACE analysis
- Dependencies installed via 'pip install -r requirements.txt' on python3, with CellProfiler 3.0.0 and the December 22 2015 Fiji release registered in paths.json

## Limitations

- Tile stitching is required and the code is optimized for the Nikon Ti-E microscope output format; pre-stitched images are not accepted in this implementation
- Data requirement: black penmarks on tiled acquisitions of the cell culture area and non-overlapping, visible ablation marks in post-MALDI microscopy are critical
- No version numbers given for Python dependencies beyond 'python3' and requirements.txt; specific pinned versions only for CellProfiler 3.0.0 and Fiji release of December 22 2015
- Co-sampling of neighboring cells by the same ablation mark occurs, but excluding such marks reduced statistical significance of markers due to lower cell numbers

## Evidence
