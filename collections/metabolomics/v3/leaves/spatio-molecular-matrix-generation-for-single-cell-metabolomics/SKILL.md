---
name: spatio-molecular-matrix-generation-for-single-cell-metabolomics
description: Use when assigning MALDI-imaging MS metabolite intensities to individual
  cultured cells from tiled pre/post-MALDI microscopy plus imzML/ibd data, to produce
  a per-cell normalized metabolic and morphological feature matrix.
when_to_use_negative:
- 'The microscopy is already pre-stitched and single-image: this implementation requires
  tile stitching and the tile-stitching code is optimized for the Nikon Ti-E output
  format (though this is a limitation of the implementation, not of the SpaceM method).'
- Ablation marks are overlapping or not visible in the post-MALDI microscopy, so local
  cell-part sampling cannot be estimated.
- The goal is bulk metabolomics or suspension/single-cell workflows not based on cultured
  cells imaged before and after MALDI ablation.
edam_operation: http://edamontology.org/operation_3715
edam_topics:
- http://edamontology.org/topic_3382
- http://edamontology.org/topic_0121
tools:
- name: SpaceM
  role: Open-source pipeline that co-registers microscopy with MALDI-IMS, normalizes
    intensities with the 45% sampling-area overlap rule, and writes the MORPHnMOL.csv
    spatio-molecular matrix.
  repo: https://github.com/alexandrovteam/SpaceM
- name: CellProfiler
  role: Runs the CP pipeline and project files that segment cells in the stitched
    pre-MALDI microscopy image; its executable and pipeline/project paths are registered
    in paths.json.
  repo: https://cellprofiler.org/previous_releases/
- name: Fiji
  role: Image-analysis dependency whose executable path is registered in paths.json
    and used by the SpaceM workflow for microscopy processing (e.g. tile stitching).
  repo: https://imagej.net/Fiji/Downloads
- name: METASPACE
  role: Web platform used to analyze the .imzML MALDI-imaging data to obtain per-mark
    molecular annotations joined to the co-registered ablation marks.
  repo: https://metaspace2020.eu/
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41592-021-01198-0
    title: SpaceM reveals metabolic states of single cells
  source_content_hash: sha256:96bb1d7166638d55f83976d0aa94c4c41f03480864a80afde7a4a396df3a4cbd
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/alexandrovteam/SpaceM
  tools:
  - CellProfiler
  - Fiji
  - METASPACE
  - Nikon Ti-E
  - Python
  - SpaceM
  edam_operation: http://edamontology.org/operation_3715
  edam_topics:
  - http://edamontology.org/topic_3382
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8666666666666667
n_claims: 15
---

# spatio-molecular-matrix-generation-for-single-cell-metabolomics

## Summary

Generates a per-cell spatio-molecular matrix (MORPHnMOL.csv) by co-registering pre/post-MALDI microscopy, segmenting cells and laser ablation marks, and joining METASPACE-derived per-mark molecular profiles to microscopy-derived phenotypic properties for in situ single-cell metabolomics of cultured cells.

## When to use

Use when assigning MALDI-imaging MS metabolite intensities to individual cultured cells from tiled pre/post-MALDI microscopy plus imzML/ibd data, to produce a per-cell normalized metabolic and morphological feature matrix.

## When NOT to use

- The microscopy is already pre-stitched and single-image: this implementation requires tile stitching and the tile-stitching code is optimized for the Nikon Ti-E output format (though this is a limitation of the implementation, not of the SpaceM method).
- Ablation marks are overlapping or not visible in the post-MALDI microscopy, so local cell-part sampling cannot be estimated.
- The goal is bulk metabolomics or suspension/single-cell workflows not based on cultured cells imaged before and after MALDI ablation.

## Inputs

- MALDI-IMS data as .RAW, .UDP, .imzML and .ibd files
- Tiled pre-MALDI microscopy images (tile_1.tif ...) with out.txt upper-left pixel x-y coordinates in um
- Tiled post-MALDI microscopy images showing visible, non-overlapping laser ablation marks, with out.txt coordinates
- CellProfiler pipeline and project files able to segment the cells
- paths.json with MF, CellProfiler pipeline, CellProfiler project, CellProfiler path, and Fiji path entries

## Outputs

- MORPHnMOL.csv spatio-molecular matrix inside the MF/Analysis/scAnalysis sub-folder, containing normalized per-cell metabolic profiles joined to microscopy-derived phenotypic properties (fluorescence intensities, morpho-spatial features)

## How to apply

Organize the main folder MF with Input/MALDI (RAW, UDP, imzML, ibd) and Input/Microscopy (preMALDI and postMALDI tiles plus out.txt tile coordinates), and register MF, CellProfiler pipeline/project, and Fiji paths in paths.json. Stitch pre- and post-MALDI microscopy tiles using the out.txt upper-left pixel x-y coordinates in um (code optimized for Nikon Ti-E output), then segment cells in the stitched pre-MALDI image with the CellProfiler pipeline and detect MALDI laser ablation marks in the post-MALDI image, co-registering them to cells via black pen-mark fiducials at subcellular precision with 45% sampling-area overlap normalization. Analyze the MALDI data on METASPACE to obtain per-mark molecular profiles, then run 'python runAnalysis.py' and follow interactive prompts; results are written to the Analysis sub-folder. Ablation marks must be visible and non-overlapping in the post-MALDI microscopy for mark detection to work.

## Related tools

- **SpaceM** (Open-source pipeline that co-registers microscopy with MALDI-IMS, normalizes intensities with the 45% sampling-area overlap rule, and writes the MORPHnMOL.csv spatio-molecular matrix.) — https://github.com/alexandrovteam/SpaceM
- **CellProfiler** (Runs the CP pipeline and project files that segment cells in the stitched pre-MALDI microscopy image; its executable and pipeline/project paths are registered in paths.json.) — https://cellprofiler.org/previous_releases/
- **Fiji** (Image-analysis dependency whose executable path is registered in paths.json and used by the SpaceM workflow for microscopy processing (e.g. tile stitching).) — https://imagej.net/Fiji/Downloads
- **METASPACE** (Web platform used to analyze the .imzML MALDI-imaging data to obtain per-mark molecular annotations joined to the co-registered ablation marks.) — https://metaspace2020.eu/

## Examples

```
python runAnalysis.py
```

## Evaluation signals

- The pipeline completes successfully and the final spatio-molecular matrix MORPHnMOL.csv is present inside the scAnalysis sub-folder of MF/Analysis.
- MORPHnMOL.csv contains one row per segmented cell with a normalized metabolic profile joined to microscopy-derived phenotypic properties (fluorescence intensities and morpho-spatial features).
- Downstream analysis (e.g. the Google Colab notebook) replicates manuscript figures from the produced matrices.
- Ablation marks in the post-MALDI microscopy are visible and non-overlapping, satisfying the stated data requirement.
- Cell-type prediction from the single-cell metabolomes in the matrix reaches the reported ~91.3% accuracy in mixed-culture benchmarks.

## Limitations

- The tile-stitching code is optimized for the Nikon Ti-E microscope output format only, and the current implementation requires stitching (pre-stitched images are not accepted in this version).
- Co-sampling of neighboring cells by the same ablation mark can occur; it minimally affects cell-type prediction accuracy in co-cultures, but excluding co-sampling marks reduces statistical significance of metabolic state markers due to lower cell numbers.
- No version numbers are given for CellProfiler, Fiji, or Python beyond CellProfiler 3.0.0 and the Fiji release of December 22 2015 specified in the README.
- METASPACE-based annotation quality depends on the data; increased sphingolipids upon NASH and the bimodality of intermediate cells remain without mechanistic explanation.

## Evidence
