---
name: scan-frequency-quality-assessment
description: 'Use when Thermo RAW-derived .FT1/.FT2 SIP data need a pre-search QC
  pass: confirming mass-range coverage, TIC continuity, and whether precursor m/z
  distributions show the mass shifts expected from isotope labeling.'
when_to_use_negative:
- Input is raw Thermo .RAW that has not yet been converted to .FT1/.FT2 by Raxport
  (the package reads FT-format scans, not vendor RAW files)
- The goal is PSM scoring or isotopic envelope simulation rather than acquisition-level
  QC — use Aerith's scoring functions instead
- Data already processed into identified PSM tables; precursor m/z frequency QC operates
  at the raw scan level before database search
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_3520
tools:
- name: Aerith
  role: R package whose QC chunks (plotScanFrequency, plotTIC, plotPrecursorMzFrequency)
    read FT1/FT2 scans and render the QC visualizations
  repo: https://github.com/thepanlab/Aerith
- name: R
  role: Runtime environment in which the Aerith package is loaded and the R Markdown
    vignette is rendered
- name: Raxport
  role: Converts Thermo RAW files into the .FT1/.FT2 scan formats consumed by Aerith's
    QC routines
  repo: https://github.com/xyz1396/Raxport.net
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1021/acs.analchem.5c03207
    title: 'Aerith: Visualization and Annotation of Isotopic Enrichment Patterns of
      Peptides and Metabolites with Stable Isotope Labeling from Proteomics and Metabolomics'
  source_content_hash: sha256:99613433e3268e6e5dfc665ac7fb6b34ae0e8bf037518854b09f05a115ae1165
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/thepanlab/Aerith
  tools:
  - Aerith
  - R
  - Raxport
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# scan-frequency-quality-assessment

## Summary

Quality-control skill from the Aerith R package that profiles FT1/FT2 mass spectrum scans by plotting scan frequency, total ion current (TIC), and precursor m/z frequency distributions, so isotope-shift artifacts and mass-range coverage in SIP experiments can be detected before downstream peptide identification.

## When to use

Use when Thermo RAW-derived .FT1/.FT2 SIP data need a pre-search QC pass: confirming mass-range coverage, TIC continuity, and whether precursor m/z distributions show the mass shifts expected from isotope labeling.

## When NOT to use

- Input is raw Thermo .RAW that has not yet been converted to .FT1/.FT2 by Raxport (the package reads FT-format scans, not vendor RAW files)
- The goal is PSM scoring or isotopic envelope simulation rather than acquisition-level QC — use Aerith's scoring functions instead
- Data already processed into identified PSM tables; precursor m/z frequency QC operates at the raw scan level before database search

## Inputs

- .FT1 (MS1) and .FT2 (MS2) mass spectrum scan files (e.g., Thermo RAW converted via Raxport)
- Aerith source tree containing inst/extdata demo FT1/FT2 files and the Aerith.Rmd vignette

## Outputs

- Rendered HTML QC vignette written to the ../doc/ directory
- Precursor m/z frequency distribution plot highlighting isotope-labeled mass shifts
- Total ion current (TIC) plot across retention time
- Scan frequency plot characterizing mass-range coverage

## How to apply

Load the Aerith package into an R session (devtools::load_all() against the Aerith source) and render the bundled 'Aerith.Rmd' vignette with rmarkdown::render("Aerith.Rmd", output_dir = "../doc/"), which executes the embedded QC chunks plotPrecursorMzFrequency, plotTIC, and plotScanFrequency on the demo FT1/FT2 files. The precursor m/z frequency plot is the key SIP-QC view: isotope-labeled peptides appear as mass-shifted populations relative to unlabeled precursors, so the distribution reveals both isotope labeling effects on precursor mass and labeling efficiency across m/z ranges. The TIC and scan-frequency plots characterize scan density and instrument coverage, exposing dropout or m/z-range gaps before running Sipros searches. Confirm the rendered HTML vignette lands in the ../doc/ output directory and that the QC figure chunks complete without errors.

## Related tools

- **Aerith** (R package whose QC chunks (plotScanFrequency, plotTIC, plotPrecursorMzFrequency) read FT1/FT2 scans and render the QC visualizations) — https://github.com/thepanlab/Aerith
- **R** (Runtime environment in which the Aerith package is loaded and the R Markdown vignette is rendered)
- **Raxport** (Converts Thermo RAW files into the .FT1/.FT2 scan formats consumed by Aerith's QC routines) — https://github.com/xyz1396/Raxport.net

## Examples

```
rmarkdown::render("Aerith.Rmd", output_dir = "../doc/")
```

## Evaluation signals

- The rendered HTML vignette exists in the ../doc/ output directory after rmarkdown::render completes
- The plotPrecursorMzFrequency, plotTIC, and plotScanFrequency chunks execute without errors and emit figures
- Unlabeled precursors cluster near theoretical monoisotopic m/z while isotope-labeled populations appear at +n isotope mass shifts in the m/z frequency plot
- TIC plot shows continuous signal across the expected retention-time span with no large scan dropout
- Scan frequency distribution spans the instrument's advertised m/z range (e.g., Orbitrap scans with charge information present in the FT files)

## Limitations

- Raxport emits .FT1/.FT2 files with charge information only for Orbitrap scans, so ion-trap scans lack charge annotation in the QC outputs
- Only MS1 and MS2 scans are supported by the FT conversion path, so the QC views do not cover higher MSn levels
- The FFT-based isotopic envelope calculation does not account for isotopic fine structure, which limits the resolution of mass-shift interpretation in the precursor m/z analysis

## Evidence

- [other] Aerith's precursor m/z frequency analysis supports quality control in SIP experiments by enabling identification of isotope-labeled peptides through their mass shifts: [evidence span withheld — anchor status: not_found]
- [methods] rmarkdown::render("Aerith.Rmd", output_dir = "../doc/"): "rmarkdown::render("Aerith.Rmd", output_dir = "../doc/")"
- [readme] Aerith is an R package that provides interfaces to read and write mass spectrum scans: "Aerith is an R package that provides interfaces to read and write mass spectrum scans, calculate the theoretical isotopic peak envelope of peptide precursors"
- [results] Aerith significantly enhances proteomic stable isotope probing (SIP) workflows by enabling systematic validation of database search outputs: "Aerith significantly enhances proteomic stable isotope probing (SIP) workflows by enabling systematic validation of database search outputs"
