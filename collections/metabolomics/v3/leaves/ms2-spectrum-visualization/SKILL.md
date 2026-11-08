---
name: ms2-spectrum-visualization
description: Use when you have prescreened HRMS extraction output (compound/precursor
  metadata, EIC traces, MS2 spectral scans) and need visual QC panels per compound,
  as in Shinyscreen's visualisation stage.
when_to_use_negative:
- You need statistical peak picking or alignment across samples rather than per-compound
  QC panels — Shinyscreen focuses exclusively on data exploration, visualisation and
  automated quality control, not full processing like xcms or RforMassSpectrometry.
- Input is raw vendor instrument output without prior extraction of EICs and MS2 scans;
  the visualisation stage expects the extraction-and-prescreening table as its input.
- Interactive instrument-native inspection is available and preferred (e.g. Xcalibur
  for Orbitrap instruments); Shinyscreen is a vendor-independent, batch-oriented prescreening
  alternative.
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: ggplot2
  role: draws the MS1 and MS2 chromatogram plots and MS2 spectra as individual plot
    objects
- name: cowplot
  role: combines the three ggplot2 objects into a cohesive stacked per-compound panel
- name: MSnbase
  role: provides the MS data structures underlying extraction of EICs and MS2 spectra
    passed to the plotting layer
- name: Shinyscreen
  role: R package/Shiny application orchestrating the visualisation stage and PDF
    report assembly
  repo: https://gitlab.com/uniluxembourg/lcsb/eci/shinyscreen
- name: R
  role: implementation language for the visualisation stage
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1186/s13321-025-01044-x
    title: 'Shinyscreen: mass spectrometry data inspection and quality checking utility'
  source_content_hash: sha256:2456985e6bbfeb757f3317e7269950d10160c9395bbdbe622fdddc5ee450a943
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
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# ms2-spectrum-visualization

## Summary

Renders per-compound MS1/MS2 chromatogram panels and representative MS2 spectra from Shinyscreen's extraction-and-prescreening output using ggplot2, combined into stacked layouts with cowplot and aggregated into a multi-page PDF QC report.

## When to use

Use when you have prescreened HRMS extraction output (compound/precursor metadata, EIC traces, MS2 spectral scans) and need visual QC panels per compound, as in Shinyscreen's visualisation stage.

## When NOT to use

- You need statistical peak picking or alignment across samples rather than per-compound QC panels — Shinyscreen focuses exclusively on data exploration, visualisation and automated quality control, not full processing like xcms or RforMassSpectrometry.
- Input is raw vendor instrument output without prior extraction of EICs and MS2 scans; the visualisation stage expects the extraction-and-prescreening table as its input.
- Interactive instrument-native inspection is available and preferred (e.g. Xcalibur for Orbitrap instruments); Shinyscreen is a vendor-independent, batch-oriented prescreening alternative.

## Inputs

- prescreened extraction table from Shinyscreen with compound/precursor metadata, EIC traces (within specified m/z tolerance), and representative MS2 spectral scans
- compound list with adduct settings used to compute precursor ion masses
- HRMS raw data readable via MSnbase/mzR (e.g. mzML)

## Outputs

- per-compound stacked panel image (MS1 EIC top, MS2 chromatogram middle, MS2 spectrum bottom, optional structure inset)
- single multi-page PDF report aggregating one combined figure per compound

## How to apply

Read the prescreened extraction table produced by Shinyscreen's extraction-and-prescreening stage, which contains precursor ion masses from the compound lists, EICs within the specified m/z tolerance, and corresponding MS2 spectral scans. For each compound entry, build separate ggplot2 plot objects: the MS1 extracted-ion chromatogram, the MS2 chromatogram, and the representative MS2 spectrum selected during prescreening. Combine the three objects into a stacked panel with cowplot (MS1 chromatogram top, MS2 chromatogram middle, MS2 spectrum bottom), optionally adding a compound-structure inset as in the Valine example. Save each panel as an image and aggregate all panels into a single multi-page PDF report, mirroring Shinyscreen's output of 'plots of MS1 and MS2 chromatograms coupled with MS2 spectra organised in a PDF report'.

## Related tools

- **ggplot2** (draws the MS1 and MS2 chromatogram plots and MS2 spectra as individual plot objects)
- **cowplot** (combines the three ggplot2 objects into a cohesive stacked per-compound panel)
- **MSnbase** (provides the MS data structures underlying extraction of EICs and MS2 spectra passed to the plotting layer)
- **Shinyscreen** (R package/Shiny application orchestrating the visualisation stage and PDF report assembly) — https://gitlab.com/uniluxembourg/lcsb/eci/shinyscreen
- **R** (implementation language for the visualisation stage)

## Examples

```
Rscript -e "library(Shinyscreen); shinyConf <- yaml::read_yaml('config.yml'); plot(extraction(shinyConf))"
```

## Evaluation signals

- The PDF report exists and contains one combined figure per compound entry in the input list.
- Each combined figure shows the MS1 chromatogram (top), MS2 chromatogram (middle) and MS2 spectrum (bottom), as in the article's example plot for Valine.
- The EIC traces correspond to precursor ion masses computed with the given adduct setting and extracted within the specified m/z tolerance.
- Each panel uses the representative MS2 spectrum selected during the prescreening/quality-assessment stage (if present for that mass).
- Plots were generated with ggplot2 and assembled with cowplot, matching the article's stated tooling for the visualisation stage.

## Limitations

- No archived code snapshot (Zenodo/OSF) is reported for Shinyscreen, so exact plotting code may need to be taken from the GitLab repository.
- Representative MS2 spectra may be absent for some input masses; panels then lack the spectrum layer.
- Visualisation depends on the quality-assessment stage's thresholds (MS1/MS2 intensity thresholds, retention time shifts, noise level); mis-set parameters propagate into misleading panels.

## Evidence

- [methods] ggplot2 and cowplot for MS1/MS2 plotting: "The ggplot2 [27] and cowplot [28] R packages are employed to generate high-quality plots of MS1 and MS2 chromatograms, as well as MS2 spectra"
- [methods] PDF report output organisation: "There are plots of MS1 and MS2 chromatograms coupled with MS2 spectra organised in a PDF report"
- [methods] extraction inputs feeding visualisation: "the precursor ion masses from the compound lists, their extracted ion chromatograms (EICs, within the specified m/z tolerance) as well as their corresponding MS2 spectral scans are retrieved"
- [methods] representative MS2 spectrum selection: "This step is also used to select a representative MS2 spectra (if present) for the input masses"
- [intro] scope: visualisation and QC only: "Its focus is exclusively on data exploration, visualisation and automated quality control combined with manual overrides"
