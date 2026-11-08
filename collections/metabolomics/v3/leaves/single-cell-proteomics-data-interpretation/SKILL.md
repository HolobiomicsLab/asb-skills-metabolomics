---
name: single-cell-proteomics-data-interpretation
description: Use when processed single-cell proteomics results (e.g., a ProteomeXchange
  dataset such as PXD037527) must be turned into reproducible figure panels that reproduce
  or match panels from a published study.
when_to_use_negative:
- Input is raw MS data that has not yet been carried through search programs and postprocessing
  — MSConnect's visualization apps consume processed results, and MSConnect is explicitly
  'not a standalone data analysis tool'
- The goal is a novel visualization outside the workflow of an existing published
  study whose panels are being reproduced
- No server environment is available — MSConnect is web-based and requires server
  deployment (e.g., via Docker), not a desktop application
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0121
tools:
- name: MSConnect
  role: Web-based platform whose Apps Marketplace visualization apps regenerate publication-ready
    panels from processed single-cell proteomics results
  repo: https://github.com/RTKlab-BYU/MSConnect
- name: Proteomics_Data_Processor
  role: Processor component that carries raw files through search/postprocessing upstream
    of visualization
  repo: https://github.com/RTKlab-BYU/Proteomics_Data_Processor
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/acs.jproteome.4c00854
    title: 'MSConnect: Open-Source, End-to-End Platform for Automated Mass Spectrometry
      Data Management, Analysis, and Visualization'
  source_content_hash: sha256:da6b59d468a75db8e56c2472c42253d6b587e1ad7f82d82f49ecf8489439eea4
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/RTKlab-BYU/MSConnect
  tools:
  - MSConnect
  - PXD037527
  - Proteomics_Data_Processor
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# single-cell-proteomics-data-interpretation

## Summary

Regenerate publication-ready visualization panels from processed single-cell proteomics results (e.g., PXD037527, DDA/DIA/WWA on an Orbitrap Exploris 480) using MSConnect's Python/HTML-based visualization apps, then compare them quantitatively and visually against the original study's figures.

## When to use

Use when processed single-cell proteomics results (e.g., a ProteomeXchange dataset such as PXD037527) must be turned into reproducible figure panels that reproduce or match panels from a published study.

## When NOT to use

- Input is raw MS data that has not yet been carried through search programs and postprocessing — MSConnect's visualization apps consume processed results, and MSConnect is explicitly 'not a standalone data analysis tool'
- The goal is a novel visualization outside the workflow of an existing published study whose panels are being reproduced
- No server environment is available — MSConnect is web-based and requires server deployment (e.g., via Docker), not a desktop application

## Inputs

- Processed single-cell proteomics results for dataset PXD037527 (deposited to ProteomeXchange via PRIDE)
- Raw MS files and metadata from DDA, DIA, and WWA acquisitions on an Orbitrap Exploris 480 (as carried through the MSConnect pipeline)
- MSConnect visualization apps (Python/HTML-based) from the MSConnect Apps Marketplace

## Outputs

- Publication-ready visualization panels exported as image files (e.g., Figure 5C panels matching the original study's Figure 4)
- A visual and quantitative concordance comparison between regenerated panels and the original published panels

## How to apply

Obtain the processed results from the deposited repository (PXD037527, deposited to ProteomeXchange via PRIDE) covering DDA, DIA, and WWA acquisition on an Orbitrap Exploris 480. Load the results into the MSConnect Server's visualization app environment (Python/HTML-based apps installed from the MSConnect Apps Marketplace, where installing a visualization app is as simple as downloading from a mobile app store). Regenerate the reported panels (Figure 5C in this paper, matching Figure 4 of the original single-cell study) using the app's statistical models and plotting routines, then export each panel as an image file. Judge success by visual and quantitative concordance of the regenerated panels against the original study's Figure 4 panels; MSConnect's traceable workflows from search programs through postprocessing and visualization are what make the regeneration repeatable.

## Related tools

- **MSConnect** (Web-based platform whose Apps Marketplace visualization apps regenerate publication-ready panels from processed single-cell proteomics results) — https://github.com/RTKlab-BYU/MSConnect
- **Proteomics_Data_Processor** (Processor component that carries raw files through search/postprocessing upstream of visualization) — https://github.com/RTKlab-BYU/Proteomics_Data_Processor

## Evaluation signals

- Regenerated Figure 5C panels visually match Figure 4 of the original single-cell proteomics manuscript
- Quantitative comparison of panel contents (data points, groupings, statistics) against the original published panels shows concordance
- Each panel was produced end-to-end from the deposited PXD037527 data through search programs, postprocessing, and visualization (traceable workflow)
- Panels were exported as standalone image files without manual data shepherding between programs

## Limitations

- Method descriptions in manuscripts are often insufficient to ensure reproducibility of complex data analysis strategies, motivating automated, traceable regeneration
- Single-cell proteomics datasets can involve hundreds or thousands of files, requiring data management systems that scale and provide transparency of all analyses
- MSConnect is not a standalone data analysis tool — it depends on third-party software for data processing/analysis, and third-party licensing must be consulted

## Evidence
