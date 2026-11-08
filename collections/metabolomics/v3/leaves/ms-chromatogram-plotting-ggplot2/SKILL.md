---
name: ms-chromatogram-plotting-ggplot2
description: Use when prescreened HRMS extraction tables containing compound metadata,
  extracted-ion chromatogram (EIC) traces and MS2 scans need to be rendered as inspection-quality
  chromatogram and spectrum plots for quality control.
when_to_use_negative:
- The goal is peak detection or quantification itself rather than visual quality inspection
  — Shinyscreen's focus is exclusively on data exploration, visualisation and automated
  quality control combined with manual overrides
- Inputs are raw vendor files without the prescreened extraction table; extraction
  and prescreening (EIC retrieval within the m/z tolerance) must run first
- Non-interactive batched statistical workflows where plotting is incidental; this
  skill targets the visualisation stage of the Shinyscreen prescreening pipeline
edam_operation: http://edamontology.org/operation_3203
edam_topics:
- http://edamontology.org/topic_0003
- http://edamontology.org/topic_3520
tools:
- name: ggplot2
  role: renders the MS1 chromatogram, MS2 chromatogram, and MS2 spectrum as individual
    plot objects
- name: cowplot
  role: combines the three plot objects into a cohesive stacked per-compound panel
- name: MSnbase
  role: upstream source of the MS/MS spectral data being plotted
- name: Shinyscreen
  role: the R/Shiny prescreening application whose visualisation stage this skill
    implements
  repo: https://gitlab.com/uniluxembourg/lcsb/eci/shinyscreen
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1186/s13321-025-01044-x
    title: 'Shinyscreen: mass spectrometry data inspection and quality checking utility'
  source_content_hash: sha256:78d1b5556700f892c09220fb35f23e8a949bcea36fa8ab3096e6f6b4de261317
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MSnbase
  - R
  - Shinyscreen
  - cowplot
  - ggplot2
  edam_operation: http://edamontology.org/operation_3203
  edam_topics:
  - http://edamontology.org/topic_0003
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# ms-chromatogram-plotting-ggplot2

## Summary

Generates publication-quality MS1 and MS2 chromatograms plus representative MS2 spectra panels for HRMS compounds, combining per-plot objects into cohesive figures via ggplot2 and cowplot as done in Shinyscreen's visualisation stage.

## When to use

Use when prescreened HRMS extraction tables containing compound metadata, extracted-ion chromatogram (EIC) traces and MS2 scans need to be rendered as inspection-quality chromatogram and spectrum plots for quality control.

## When NOT to use

- The goal is peak detection or quantification itself rather than visual quality inspection — Shinyscreen's focus is exclusively on data exploration, visualisation and automated quality control combined with manual overrides
- Inputs are raw vendor files without the prescreened extraction table; extraction and prescreening (EIC retrieval within the m/z tolerance) must run first
- Non-interactive batched statistical workflows where plotting is incidental; this skill targets the visualisation stage of the Shinyscreen prescreening pipeline

## Inputs

- prescreened extraction table with compound/precursor metadata (compound list or mass list based)
- extracted ion chromatogram (EIC) traces within the specified m/z tolerance
- representative MS2 spectral scans selected during prescreening

## Outputs

- per-compound combined panel: MS1 chromatogram (top), MS2 chromatogram (middle), MS2 spectrum (bottom)
- multi-page PDF report with one combined figure per compound

## How to apply

Read the extraction-and-prescreening table produced upstream (compound/precursor metadata, EIC traces within the specified m/z tolerance, and representative MS2 spectral scans). For each compound entry, build the MS1 extracted-ion chromatogram, the MS2 chromatogram, and the representative MS2 spectrum as separate ggplot2 plot objects. Combine the three objects into a stacked layout with cowplot, optionally adding a compound-structure inset as Shinyscreen does for Valine. Save each per-compound panel and aggregate all panels into a single multi-page PDF report; the rationale is that ggplot2/cowplot give high-quality plots of MS1 and MS2 chromatograms as well as MS2 spectra in a vendor-independent, open-source manner.

## Related tools

- **ggplot2** (renders the MS1 chromatogram, MS2 chromatogram, and MS2 spectrum as individual plot objects)
- **cowplot** (combines the three plot objects into a cohesive stacked per-compound panel)
- **MSnbase** (upstream source of the MS/MS spectral data being plotted)
- **Shinyscreen** (the R/Shiny prescreening application whose visualisation stage this skill implements) — https://gitlab.com/uniluxembourg/lcsb/eci/shinyscreen

## Evaluation signals

- Each compound in the input table yields exactly one combined panel in the PDF report
- Panel layout matches the article example: MS1 chromatogram top, MS2 chromatogram middle, MS2 spectrum bottom
- EIC traces correspond to the precursor ion masses calculated from the compound list adduct setting, extracted within the specified m/z tolerance
- Only MS2 scans selected as representative during prescreening are drawn, not all scans
- Output is a single multi-page PDF with one figure per compound, as in the article's Valine example

## Limitations

- Plots depend on the quality of the upstream extraction step (EIC and MS2 retrieval within the m/z tolerance); poor extraction produces poor panels
- Representative MS2 spectra may be absent for some masses, in which case the spectrum sub-panel is empty or omitted
- Visualisation is for inspection/QC, not for statistical filtering — Shinyscreen's filtering by m/z, retention time or quality score is a separate prescreening step

## Evidence

- [methods] ggplot2 and cowplot used for high-quality plots of MS1 and MS2 chromatograms and MS2 spectra: "The ggplot2 [27] and cowplot [28] R packages are employed to generate high-quality plots of MS1 and MS2 chromatograms, as well as MS2 spectra"
- [methods] EIC retrieval within specified m/z tolerance: "the precursor ion masses from the compound lists, their extracted ion chromatograms (EICs, within the specified m/z tolerance) as well as their corresponding MS2 spectral scans are retrieved"
- [methods] Visualisation is a core Shinyscreen step: "The core steps of Shinyscreen include initialisation, extraction and prescreening, visualisation and integration with MetFrag for further analysis"
- [methods] PDF report organisation: "There are plots of MS1 and MS2 chromatograms coupled with MS2 spectra organised in a PDF report"
- [methods] Representative MS2 spectrum selection: "This step is also used to select a representative MS2 spectra (if present) for the input masses"
- [intro] Shinyscreen scope: exploration, visualisation, automated QC: "Its focus is exclusively on data exploration, visualisation and automated quality control combined with manual overrides"
