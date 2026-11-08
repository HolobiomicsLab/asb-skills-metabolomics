---
name: maldi-metabolite-annotation-via-metaspace
description: Use when raw MALDI-imaging MS acquisitions (.RAW, .UDP, .imzML, .ibd)
  from cultured-cell experiments must be metabolite-annotated before per-cell spatio-molecular
  analysis with SpaceM.
when_to_use_negative:
- Input is already a per-cell feature/spatio-molecular matrix (MORPHnMOL.csv) — annotation
  has already happened upstream
- Data are bulk LC-MS/MS measurements (e.g., the ND vs WD mouse liver validation tissue)
  rather than MALDI-imaging acquisitions
- Post-MALDI microscopy lacks visible or non-overlapping ablation marks, making per-cell
  metabolite assignment unreliable
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: METASPACE
  role: Web platform used to analyze/annotate the MALDI-imaging (.RAW/.UDP/.imzML/.ibd)
    data with metabolite identifications before the SpaceM pipeline run
  repo: https://metaspace2020.eu/
- name: SpaceM
  role: Downstream open-source pipeline that consumes the annotated MALDI data plus
    microscopy to build the per-cell spatio-molecular matrix
  repo: github.com/alexandrovteam/SpaceM
- name: CellProfiler
  role: Provides the segmentation pipeline/project (registered in paths.json) whose
    cell masks receive metabolite intensities from annotated ablation marks
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41592-021-01198-0
    title: SpaceM reveals metabolic states of single cells
  source_content_hash: sha256:d0b700006e369416fe8634e41e03bbf62c866c6dfc188c5f27d679e3475de1fb
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
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 21
---

# maldi-metabolite-annotation-via-metaspace

## Summary

Annotate MALDI-imaging mass spectrometry metabolite identities with METASPACE (metaspace2020.eu) as the mandatory preprocessing step of a SpaceM single-cell metabolomics run, turning raw .RAW/.UDP/.imzML/.ibd acquisitions into metabolite-annotated ion images whose intensities are later assigned to individual cells.

## When to use

Use when raw MALDI-imaging MS acquisitions (.RAW, .UDP, .imzML, .ibd) from cultured-cell experiments must be metabolite-annotated before per-cell spatio-molecular analysis with SpaceM.

## When NOT to use

- Input is already a per-cell feature/spatio-molecular matrix (MORPHnMOL.csv) — annotation has already happened upstream
- Data are bulk LC-MS/MS measurements (e.g., the ND vs WD mouse liver validation tissue) rather than MALDI-imaging acquisitions
- Post-MALDI microscopy lacks visible or non-overlapping ablation marks, making per-cell metabolite assignment unreliable

## Inputs

- MALDI-imaging acquisition files: *.RAW, *.UDP, *.imzML, *.ibd
- post-MALDI microscopy tile images (tile_1.tif ...) with out.txt tile-coordinate files
- paths.json registration of MF, CellProfiler pipeline/project, and Fiji paths

## Outputs

- METASPACE metabolite annotations for the MALDI-imaging dataset
- Annotated ion intensity data feeding the SpaceM spatio-molecular matrix MORPHnMOL.csv in MF/Analysis/scAnalysis

## How to apply

Collect the four required MALDI acquisition files (.RAW, .UDP, .imzML, .ibd) into the MF/Input/MALDI folder of the SpaceM main folder. Before annotation is meaningful, confirm that laser ablation marks are visible and non-overlapping in the post-MALDI microscopy tiles, since SpaceM co-registers ablation marks with cell microscopy images to assign metabolite intensities to cells. Submit the imzML/ibd dataset to METASPACE (https://metaspace2020.eu/) for metabolite identification, then register the MF path, CellProfiler path and pipeline/project paths, and Fiji path in paths.json and execute 'python runAnalysis.py' so annotated MALDI data are stitched (Nikon Ti-E tile format), co-registered, and segmented. The annotated, per-cell intensities ultimately land in MORPHnMOL.csv under MF/Analysis/scAnalysis.

## Related tools

- **METASPACE** (Web platform used to analyze/annotate the MALDI-imaging (.RAW/.UDP/.imzML/.ibd) data with metabolite identifications before the SpaceM pipeline run) — https://metaspace2020.eu/
- **SpaceM** (Downstream open-source pipeline that consumes the annotated MALDI data plus microscopy to build the per-cell spatio-molecular matrix) — github.com/alexandrovteam/SpaceM
- **CellProfiler** (Provides the segmentation pipeline/project (registered in paths.json) whose cell masks receive metabolite intensities from annotated ablation marks)

## Evaluation signals

- All four MALDI file types (*.RAW, *.UDP, *.imzML, *.ibd) are present in MF/Input/MALDI before annotation
- Ablation marks are verified as visible and non-overlapping in the post-MALDI microscopy tiles
- Dataset is successfully processed/annotated on METASPACE (https://metaspace2020.eu/)
- Downstream check: MORPHnMOL.csv exists in MF/Analysis/scAnalysis after runAnalysis.py completes without prompting errors

## Limitations

- Tile stitching is optimized for the Nikon Ti-E microscope output format; pre-stitched images are not accepted in this implementation
- Unresolved or overlapping ablation marks in post-MALDI microscopy undermine per-cell metabolite assignment
- Version numbers for the surrounding software stack are unspecified in the article methods

## Evidence
