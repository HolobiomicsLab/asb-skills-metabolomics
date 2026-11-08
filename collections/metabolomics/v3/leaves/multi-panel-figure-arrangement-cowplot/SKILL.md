---
name: multi-panel-figure-arrangement-cowplot
description: Use when separate ggplot2 plot objects (e.g. MS1 EIC trace, MS2 EIC trace,
  representative MS2 spectrum for one compound) must be vertically stacked into one
  publication-quality panel and exported to a PDF report.
when_to_use_negative:
- Input is already a finished multi-panel figure or a feature/quantification table
  with no chromatographic or spectral traces to plot
- The goal is statistical QC scoring (noise level, retention-time shift, MS1/MS2 intensity
  thresholds) rather than figure composition — that belongs to the quality-assessment
  stage, not visualisation
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3318
tools:
- name: ggplot2
  role: Draws the individual MS1 chromatogram, MS2 chromatogram and MS2 spectrum plot
    objects before combination
- name: cowplot
  role: Combines the three ggplot2 plot objects into a single stacked per-compound
    panel layout
- name: Shinyscreen
  role: Hosts the visualisation workflow stage that produces the PDF report of combined
    MS1/MS2 figures
  repo: https://gitlab.com/uniluxembourg/lcsb/eci/shinyscreen
- name: MSnbase
  role: Source of the extracted HRMS peak/EIC/MS2-scan data that feed the plots
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1186/s13321-025-01044-x
    title: 'Shinyscreen: mass spectrometry data inspection and quality checking utility'
  source_content_hash: sha256:25325c3b7e62d199ec47556e5990d2af5009d0beb5c5971568d05056b6ae3d58
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
  - http://edamontology.org/topic_3318
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# multi-panel-figure-arrangement-cowplot

## Summary

Composes per-compound mass spectrometry figures by building individual ggplot2 plot objects (MS1 chromatogram, MS2 chromatogram, MS2 spectrum) and stacking them into a single cohesive panel with the cowplot R package, as implemented in Shinyscreen's visualisation stage.

## When to use

Use when separate ggplot2 plot objects (e.g. MS1 EIC trace, MS2 EIC trace, representative MS2 spectrum for one compound) must be vertically stacked into one publication-quality panel and exported to a PDF report.

## When NOT to use

- Input is already a finished multi-panel figure or a feature/quantification table with no chromatographic or spectral traces to plot
- The goal is statistical QC scoring (noise level, retention-time shift, MS1/MS2 intensity thresholds) rather than figure composition — that belongs to the quality-assessment stage, not visualisation

## Inputs

- prescreened extraction table with compound/precursor metadata, m/z and retention-time values
- extracted ion chromatograms (EICs) within the specified m/z tolerance
- representative MS2 spectral scans selected during prescreening
- optional compound structure graphic for the panel inset

## Outputs

- per-compound stacked panel image (MS1 chromatogram top, MS2 chromatogram middle, MS2 spectrum bottom)
- multi-page PDF report containing one combined figure per compound

## How to apply

Read the prescreened extraction table containing compound/precursor metadata, EIC traces retrieved within the specified m/z tolerance, and representative MS2 spectral scans. For each compound entry, draw the MS1 extracted-ion chromatogram, the MS2 chromatogram, and the representative MS2 spectrum as three separate ggplot2 plot objects. Combine them into a stacked layout with cowplot (e.g. plot_grid with ncol = 1, MS1 chromatogram on top, MS2 chromatogram in the middle, MS2 spectrum on the bottom), optionally adding a compound-structure inset as in the Valine example. Save each per-compound panel as an image and aggregate all panels into a single multi-page PDF report, mirroring Shinyscreen's Output stage. Rationale: cowplot handles consistent alignment and labelling of ggplot2 objects, which keeps the three plot types visually coherent across thousands of prescreened peaks.

## Related tools

- **ggplot2** (Draws the individual MS1 chromatogram, MS2 chromatogram and MS2 spectrum plot objects before combination)
- **cowplot** (Combines the three ggplot2 plot objects into a single stacked per-compound panel layout)
- **Shinyscreen** (Hosts the visualisation workflow stage that produces the PDF report of combined MS1/MS2 figures) — https://gitlab.com/uniluxembourg/lcsb/eci/shinyscreen
- **MSnbase** (Source of the extracted HRMS peak/EIC/MS2-scan data that feed the plots)

## Examples

```
panel <- cowplot::plot_grid(p_ms1_chrom, p_ms2_chrom, p_ms2_spec, ncol = 1); ggplot2::ggsave("valine_panel.pdf", panel)
```

## Evaluation signals

- Each compound yields exactly one combined figure with three panels in the fixed order: MS1 chromatogram (top), MS2 chromatogram (middle), MS2 spectrum (bottom)
- The output PDF report exists and contains one combined figure per compound, matching the article's Valine example layout
- Chromatogram traces fall within the specified m/z tolerance used during EIC extraction
- All panels in a figure share consistent axes, fonts and alignment after cowplot composition (no clipped labels or misaligned strips)
- Compounds with no MS2 data still render an MS1 panel or a clearly empty MS2 slot rather than failing silently

## Limitations

- The article documents the visualisation approach qualitatively (ggplot2 + cowplot) but does not report specific theme settings, panel dimensions, or cowplot parameters, so exact layout tuning must be derived from the GitLab source
- Panels depend on upstream prescreening quality: if MS2 spectral scans were not retrieved or the representative spectrum selection was empty, the corresponding panel cannot be populated
- No archived code snapshot (Zenodo/OSF) is reported, so reproducibility of the exact figure styling relies on the live GitLab repository state

## Evidence

- [methods] ggplot2/cowplot plotting: "The ggplot2 [27] and cowplot [28] R packages are employed to generate high-quality plots of MS1 and MS2 chromatograms, as well as MS2 spectra"
- [methods] PDF report output: "There are plots of MS1 and MS2 chromatograms coupled with MS2 spectra organised in a PDF report"
- [intro] scope of Shinyscreen visualisation: "Its focus is exclusively on data exploration, visualisation and automated quality control combined with manual overrides"
