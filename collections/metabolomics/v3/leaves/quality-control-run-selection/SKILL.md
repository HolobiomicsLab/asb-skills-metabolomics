---
name: quality-control-run-selection
description: Use when raw targeted LC-MS runs (e.g. mzML) need a pre-check that each
  target peak is present within its expected m/z and retention time window before
  committing to full peak detection and integration.
when_to_use_negative:
- The goal is full peak detection and integration with quality metrics, not a pre-run
  visual screening of target visibility — run TARDIS in normal (non-screening) mode
  instead
- Input is already preprocessed/peak-picked data or a feature table rather than raw
  LC-MS runs, since EIC extraction requires raw chromatographic data
- Untargeted preprocessing is needed — TARDIS is designed for targeted integration,
  and tools like xcms, MZmine or OpenMS are the appropriate alternative
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: TARDIS
  role: Open-source R package whose screening_mode = TRUE argument performs targeted
    EIC extraction and diagnostic plotting on auto-selected QC runs
  repo: https://github.com/UGent-LIMET/TARDIS
- name: R
  role: Execution environment (version >= 4.4.0) in which the TARDIS package is installed
    via BiocManager and run
  repo: https://cloud.r-project.org/index.html
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/acs.analchem.5c00567
    title: Automated Integration and Quality Assessment of Chromatographic Peaks in
      LC–MS-Based Metabolomics and Lipidomics Using TARDIS
  source_content_hash: sha256:8ace3d59326a46b7aa6a93f74c765fc77c79bbe41010ad1cb474bde3ed67716f
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/UGent-LIMET/TARDIS
  repo_url: https://github.com/UGent-LIMET/TARDIS
  tools:
  - R
  - TARDIS
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# quality-control-run-selection

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A TARDIS (TArgeted Raw Data Integration In Spectrometry) R-package workflow in which QC runs of targeted LC-MS metabolomics/lipidomics data are automatically selected for screening, and extracted ion chromatograms (EICs) are rendered as diagnostic plots (five EICs per image) so target peak visibility can be verified before full peak integration.

## When to use

Use when raw targeted LC-MS runs (e.g. mzML) need a pre-check that each target peak is present within its expected m/z and retention time window before committing to full peak detection and integration.

## When NOT to use

- The goal is full peak detection and integration with quality metrics, not a pre-run visual screening of target visibility — run TARDIS in normal (non-screening) mode instead
- Input is already preprocessed/peak-picked data or a feature table rather than raw LC-MS runs, since EIC extraction requires raw chromatographic data
- Untargeted preprocessing is needed — TARDIS is designed for targeted integration, and tools like xcms, MZmine or OpenMS are the appropriate alternative

## Inputs

- Raw targeted LC-MS runs (mzML files), e.g. the example mzML files bundled with the TARDIS R package
- Target list defining expected m/z and retention time windows per analyte

## Outputs

- Diagnostic EIC plot images saved in a designated output folder, five EICs per image, restricted to the automatically selected QC runs
- Confirmed target-visibility status per analyte prior to full peak integration

## How to apply

Load the TARDIS R package in R (>= 4.4.0) and run targeted peak integration via tardisPeaks() with screening_mode = TRUE, so that QC runs within the input LC-MS runs are automatically selected for screening rather than every sample. TARDIS generates extracted ion chromatograms for each target and saves the resulting EIC plots in a designated output folder, grouped with five EICs per image. Inspect these images to confirm each target peak is visible inside the expected m/z and retention time window, comparing against the corresponding figures in the TARDIS quick-start vignette. Only once targets are confirmed visible should full integration with quality metrics (area under the curve, maximum intensity, points over the peak, custom signal-to-noise ratio, bell-curve similarity) be run.

## Related tools

- **TARDIS** (Open-source R package whose screening_mode = TRUE argument performs targeted EIC extraction and diagnostic plotting on auto-selected QC runs) — https://github.com/UGent-LIMET/TARDIS
- **R** (Execution environment (version >= 4.4.0) in which the TARDIS package is installed via BiocManager and run) — https://cloud.r-project.org/index.html

## Examples

```
tardisPeaks(screening_mode = TRUE)
```

## Evaluation signals

- EIC diagnostic plot image files exist in the designated output folder after tardisPeaks(screening_mode = TRUE) completes
- Each plot image contains five EICs, matching the vignette layout
- Only QC runs appear in the screening plots, confirming QC-run auto-selection worked
- EIC peaks fall within the expected m/z and retention time windows for their targets
- Generated plots visually match the corresponding figures shown in the TARDIS quick-start vignette

## Limitations

- Screening selects QC runs automatically; non-QC runs are not screened, so sample-specific interference outside QC runs is not checked
- Package data download can hit R's default connection timeout; the README recommends raising it, e.g. options(timeout = "300")
- Requires R >= 4.4.0 and BiocManager version 3.20, plus Rtools (Windows) or Xcode/GNU Fortran (Mac) to compile

## Evidence

- [readme] Make sure `R` (**version >= 4.4.0**) is installed on your computer:: [evidence span withheld — anchor status: external_doc]
- [readme] Since the package contains some example data, connection timeout is often an issue.: [evidence span withheld — anchor status: external_doc]
