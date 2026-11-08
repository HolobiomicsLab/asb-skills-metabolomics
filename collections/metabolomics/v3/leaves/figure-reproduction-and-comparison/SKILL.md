---
name: figure-reproduction-and-comparison
description: 'Use when a previously published MS proteomics study has deposited processed
  results (e.g. PXD037527: DDA/DIA/WWA on Orbitrap Exploris 480) and the goal is to
  reproduce its figures end-to-end.'
when_to_use_negative:
- You only have raw vendor files and no deposited processed results — MSConnect 'is
  not a standalone data analysis tool. It depends on third-party software for data
  processing/analysis'
- The goal is novel exploratory visualization rather than verifying that a workflow
  reproduces a published figure
- Third-party processing/visualization modules required for the panel type are not
  licensed or not available in the marketplace (licensing must be consulted with their
  respective owners)
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3391
tools:
- name: MSConnect
  role: Hosts the visualization app environment and automated workflow that carries
    raw files through search programs, postprocessing, and visualization to regenerate
    the panels
  repo: https://github.com/RTKlab-BYU/MSConnect
- name: Proteomics_Data_Processor
  role: Upstream processing tool used with MSConnect to carry raw files through analysis
    before visualization
  repo: https://github.com/RTKlab-BYU/Proteomics_Data_Processor
- name: Raw_File_Uploader
  role: Uploads raw MS files and metadata to the MSConnect database, ensuring traceability
    of files through the reproduction workflow
  repo: https://github.com/RTKlab-BYU/Raw_File_Uploader
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/acs.jproteome.4c00854
    title: 'MSConnect: Open-Source, End-to-End Platform for Automated Mass Spectrometry
      Data Management, Analysis, and Visualization'
  source_content_hash: sha256:46241ea2d01d6e4efbff0d7e0cf72a8ef72b62983f840a81a8417359904bd1db
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/RTKlab-BYU/MSConnect
  tools:
  - MSConnect
  - PXD037527
  - Proteomics_Data_Processor
  - Raw_File_Uploader
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3391
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# figure-reproduction-and-comparison

## Summary

Regenerate publication-ready visualization panels from deposited processed single-cell proteomics data using MSConnect's preconfigured visualization apps, then quantitatively and visually compare the regenerated panels against the corresponding figures of the original published study to establish workflow reproducibility.

## When to use

Use when a previously published MS proteomics study has deposited processed results (e.g. PXD037527: DDA/DIA/WWA on Orbitrap Exploris 480) and the goal is to reproduce its figures end-to-end.

## When NOT to use

- You only have raw vendor files and no deposited processed results — MSConnect 'is not a standalone data analysis tool. It depends on third-party software for data processing/analysis'
- The goal is novel exploratory visualization rather than verifying that a workflow reproduces a published figure
- Third-party processing/visualization modules required for the panel type are not licensed or not available in the marketplace (licensing must be consulted with their respective owners)

## Inputs

- Processed single-cell proteomics results for dataset PXD037527 from the ProteomeXchange/PRIDE repository (DDA, DIA, and WWA acquisition on an Orbitrap Exploris 480)
- MSConnect visualization app from the Apps Marketplace (Python/HTML-based plotting and statistical routines)
- Original published manuscript figure (Figure 4 of the single-cell proteomics study) as comparison reference

## Outputs

- Regenerated Figure 5C visualization panels exported as image files
- Visual and quantitative concordance assessment of regenerated panels vs. original Figure 4 panels

## How to apply

Obtain the deposited processed results from the ProteomeXchange/PRIDE repository (dataset identifier PXD037527) and load them into the MSConnect Server's Python/HTML-based visualization app environment from the MSConnect Apps Marketplace. Install the relevant visualization app, which per the paper is 'as simple as downloading a new app from a mobile app store'. Run the app's statistical models and plotting routines so the raw files are carried through search programs, postprocessing, and visualization without manual shepherding, and export each regenerated panel (the Figure 5C panels) as an image file. Finally, compare the exported panels against the original manuscript's Figure 4 panels both visually and quantitatively to assess concordance.

## Related tools

- **MSConnect** (Hosts the visualization app environment and automated workflow that carries raw files through search programs, postprocessing, and visualization to regenerate the panels) — https://github.com/RTKlab-BYU/MSConnect
- **Proteomics_Data_Processor** (Upstream processing tool used with MSConnect to carry raw files through analysis before visualization) — https://github.com/RTKlab-BYU/Proteomics_Data_Processor
- **Raw_File_Uploader** (Uploads raw MS files and metadata to the MSConnect database, ensuring traceability of files through the reproduction workflow) — https://github.com/RTKlab-BYU/Raw_File_Uploader

## Evaluation signals

- Regenerated Figure 5C panels match Figure 4 of the original single-cell proteomics manuscript when compared side by side
- Quantitative comparison of panel values between regenerated and original figures shows concordance, not just visual similarity
- Traceability: raw files can be followed through search programs, postprocessing, and visualization within MSConnect rather than manual transfer
- Each panel is exported as an image file and reproducible by re-running the same app on the same deposited dataset (PXD037527)

## Limitations

- MSConnect is not a standalone analysis tool — third-party processing and visualization software is not included and licensing requirements must be consulted with their respective owners
- Visualization depends on the user workflow and may require 3rd party modules
- Method descriptions in manuscripts are often insufficient to ensure reproducibility of complex data analysis strategies, complicating panel-by-panel comparisons
- Single-cell and spatial proteomics data sets can involve hundreds or thousands of files, so the comparison must be managed on a system that scales

## Evidence

- [results] MSConnect automatically produced the Figure 5C panels, which match Figure 4 of the original single-cell proteomics manuscript: [evidence span withheld — anchor status: not_found]
- [methods] installing a wrapper or visualization app is as simple as downloading a new app from a mobile app store: "installing a wrapper or visualization app is as simple as downloading a new app from a mobile app store"
- [readme] Result Interpretation: Generates tables and figures for easy interpretation and presentation of results: "Result Interpretation: Generates tables and figures for easy interpretation and presentation of results."
- [abstract] Manually shepherding data across multiple programs and in-house-developed scripts can be error prone and labor intensive: "Manually shepherding data across multiple programs and in-house-developed scripts can be error prone and labor intensive."
- [readme] MSConnect is not a standalone data analysis tool. It depends on third-party software for data processing/analysis: "MSConnect is not a standalone data analysis tool. It depends on third-party software for data processing/analysis"
