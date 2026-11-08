---
name: pdf-report-generation-for-ms-quality-control
description: Use when prescreened HRMS extraction results (compound/precursor metadata,
  EIC traces, representative MS2 spectra) must be rendered into per-compound figure
  panels and aggregated into a PDF QC report.
when_to_use_negative:
- Raw vendor HRMS files have not yet been extracted — Shinyscreen's initialisation
  and extraction-and-prescreening stages (EIC retrieval within the m/z tolerance,
  MS2 scan selection) must run first
- The goal is interactive, point-and-click data exploration rather than a batch-generated
  report — Shinyscreen also operates as a self-documented Shiny web GUI for that purpose
- Deep, programmatic analysis of MS data beyond prescreening visualisation is needed
  — low-level packages such as mzR or MSnbase directly, or platforms like xcms, RforMassSpectrometry
  or patRoon, are more appropriate
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3520
tools:
- name: ggplot2
  role: R plotting package used to generate the high-quality MS1 and MS2 chromatogram
    plots and MS2 spectrum plots for each compound panel
- name: cowplot
  role: R package used to combine the three ggplot2 plot objects into a cohesive stacked
    per-compound layout
- name: MSnbase
  role: Underlying R infrastructure from which MS1/MS2 chromatogram and spectral data
    are retrieved for visualisation
- name: Shinyscreen
  role: Encapsulates the visualisation workflow step that turns prescreened extraction
    output into the per-compound plots and the aggregated PDF report
  repo: https://gitlab.com/uniluxembourg/lcsb/eci/shinyscreen
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1186/s13321-025-01044-x
    title: 'Shinyscreen: mass spectrometry data inspection and quality checking utility'
  source_content_hash: sha256:ca4083449d3de48a02bcb3d747c1ba5c0ae80ba73926d5b56700c846bd12a891
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MSnbase
  - R
  - Shinyscreen
  - cowplot
  - ggplot2
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# pdf-report-generation-for-ms-quality-control

## Summary

Renders per-compound MS1/MS2 chromatogram and MS2 spectrum figure panels from prescreened HRMS extraction results using ggplot2 and cowplot in R, then aggregates the panels into a multi-page PDF quality-control report as done in Shinyscreen.

## When to use

Use when prescreened HRMS extraction results (compound/precursor metadata, EIC traces, representative MS2 spectra) must be rendered into per-compound figure panels and aggregated into a PDF QC report.

## When NOT to use

- Raw vendor HRMS files have not yet been extracted — Shinyscreen's initialisation and extraction-and-prescreening stages (EIC retrieval within the m/z tolerance, MS2 scan selection) must run first
- The goal is interactive, point-and-click data exploration rather than a batch-generated report — Shinyscreen also operates as a self-documented Shiny web GUI for that purpose
- Deep, programmatic analysis of MS data beyond prescreening visualisation is needed — low-level packages such as mzR or MSnbase directly, or platforms like xcms, RforMassSpectrometry or patRoon, are more appropriate

## Inputs

- Prescreened extraction table with compound/precursor metadata, EIC traces and representative MS2 spectra (from the Shinyscreen extraction-and-prescreening stage)
- MS1 extracted-ion chromatogram data for each target mass/compound
- MS2 chromatogram data and representative MS2 spectral scans per precursor ion mass

## Outputs

- Per-compound combined figure panel image (MS1 chromatogram top, MS2 chromatogram middle, MS2 spectrum bottom)
- Single multi-page PDF quality-control report containing one combined figure per compound

## How to apply

Read the prescreened extraction table produced by the extraction-and-prescreening stage, which holds compound/precursor metadata, extracted ion chromatograms (EICs retrieved within the specified m/z tolerance) and representative MS2 spectral scans. For each compound entry, build three ggplot2 plot objects: the MS1 extracted-ion chromatogram, the MS2 chromatogram, and the representative MS2 spectrum. Combine the three plot objects into a cohesive stacked layout (MS1 chromatogram on top, MS2 chromatogram in the middle, MS2 spectrum at the bottom) using cowplot. Optionally add a compound-structure inset to the panel, as shown for Valine in the article's example. Save each per-compound panel as an image and aggregate all panels into a single multi-page PDF report, so the QC stage is followed by a human-readable artifact that supports upload of spectra to public data resources.

## Related tools

- **ggplot2** (R plotting package used to generate the high-quality MS1 and MS2 chromatogram plots and MS2 spectrum plots for each compound panel)
- **cowplot** (R package used to combine the three ggplot2 plot objects into a cohesive stacked per-compound layout)
- **MSnbase** (Underlying R infrastructure from which MS1/MS2 chromatogram and spectral data are retrieved for visualisation)
- **Shinyscreen** (Encapsulates the visualisation workflow step that turns prescreened extraction output into the per-compound plots and the aggregated PDF report) — https://gitlab.com/uniluxembourg/lcsb/eci/shinyscreen

## Evaluation signals

- The PDF report file exists and contains one combined figure per compound entry
- Each figure shows the MS1 chromatogram (top), MS2 chromatogram (middle) and MS2 spectrum (bottom), mirroring the article's example plot (e.g. Valine)
- Panels are built from ggplot2 plot objects combined with cowplot, matching the article's stated tooling
- Plotted chromatograms correspond to EICs retrieved within the specified m/z tolerance for the precursor ion masses in the compound list
- All per-compound panels are aggregated into a single multi-page PDF rather than scattered individual files

## Limitations

- The visualisation step depends on upstream extraction and prescreening quality: it renders only what was retrieved (EICs within the m/z tolerance and available MS2 spectral scans), so compounds lacking MS2 data cannot show a spectrum panel
- Only the plotting workflow is documented at this level — concrete plot parameterisation (colour scales, axes, dimensions) is not specified in the article
- No archived code snapshot (Zenodo/OSF) is reported, so the exact rendering implementation must be inspected in the GitLab repository

## Evidence

- [methods] ggplot2/cowplot tooling for MS1/MS2 plotting: "The ggplot2 [27] and cowplot [28] R packages are employed to generate high-quality plots of MS1 and MS2 chromatograms, as well as MS2 spectra"
- [methods] PDF report output format: "There are plots of MS1 and MS2 chromatograms coupled with MS2 spectra organised in a PDF report"
- [methods] EIC retrieval within m/z tolerance feeding the plots: "the precursor ion masses from the compound lists, their extracted ion chromatograms (EICs, within the specified m/z tolerance) as well as their corresponding MS2 spectral scans are retrieved from the"
