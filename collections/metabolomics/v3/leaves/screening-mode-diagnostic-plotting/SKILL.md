---
name: screening-mode-diagnostic-plotting
description: Use when setting up a targeted LC-MS metabolomics/lipidomics assay in
  TARDIS and you need to verify, before peak detection, that each target's peak is
  actually visible within its expected m/z and RT window in QC runs.
when_to_use_negative:
- The goal is final quantitative peak integration (AUC extraction per target per run)
  rather than pre-run visual QC — run full TARDIS integration instead
- Input data is already preprocessed (feature table or peak table), since screening
  mode operates directly on raw mzML chromatograms
edam_operation: http://edamontology.org/operation_3203
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3172
tools:
- name: TARDIS
  role: R package whose screening_mode = TRUE run of tardisPeaks renders the diagnostic
    EIC plots to the output folder
  repo: https://github.com/UGent-LIMET/TARDIS
- name: R
  role: Runtime environment (>= 4.4.0) in which the TARDIS package is installed and
    executed
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/acs.analchem.5c00567
    title: Automated Integration and Quality Assessment of Chromatographic Peaks in
      LC–MS-Based Metabolomics and Lipidomics Using TARDIS
  source_content_hash: sha256:f8740c59770385cc12ae598c8ba4fd9c90023a5e502f276ad73013cd4d2295df
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
  edam_operation: http://edamontology.org/operation_3203
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# screening-mode-diagnostic-plotting

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Generates extracted ion chromatogram (EIC) diagnostic plots with TARDIS's screening mode (tardisPeaks with screening_mode = TRUE) so a user can visually confirm that target peaks are visible within their expected m/z and retention time windows in QC runs before committing to full targeted peak integration.

## When to use

Use when setting up a targeted LC-MS metabolomics/lipidomics assay in TARDIS and you need to verify, before peak detection, that each target's peak is actually visible within its expected m/z and RT window in QC runs.

## When NOT to use

- The goal is final quantitative peak integration (AUC extraction per target per run) rather than pre-run visual QC — run full TARDIS integration instead
- Input data is already preprocessed (feature table or peak table), since screening mode operates directly on raw mzML chromatograms

## Inputs

- Raw LC-MS runs as mzML files (e.g., the example mzML files bundled with the TARDIS package)
- Target list defining expected m/z and retention time windows per compound

## Outputs

- Diagnostic EIC plot images (five EICs per image) for the selected QC runs, saved in the output folder
- Visual confirmation per target of whether a peak is visible in the expected m/z/RT window

## How to apply

Load the mzML files bundled with the TARDIS R package (installed from GitHub, requires R >= 4.4.0) and run targeted peak integration via tardisPeaks(screening_mode = TRUE), which automatically selects the QC runs for screening. TARDIS then renders diagnostic EIC plots for the targeted m/z and retention-time windows of those QC runs, arranging five EICs per image, and saves the images to the designated output folder. Open the saved EIC images and inspect them — confirming a visible peak inside each target's expected window means the target list and RT/m/z windows are correctly specified before running full peak integration.

## Related tools

- **TARDIS** (R package whose screening_mode = TRUE run of tardisPeaks renders the diagnostic EIC plots to the output folder) — https://github.com/UGent-LIMET/TARDIS
- **R** (Runtime environment (>= 4.4.0) in which the TARDIS package is installed and executed)

## Examples

```
library(TARDIS); tardisPeaks(mzml_files, targets, screening_mode = TRUE)
```

## Evaluation signals

- EIC image files exist in the designated output folder after the screening-mode run
- Each saved image contains five EIC plots and corresponds visually to the matching figure in the TARDIS vignette (https://pablovgd.github.io/TARDIS/)
- Plots are generated only for QC runs selected automatically by screening mode
- Each plotted EIC spans the target's expected m/z and retention time window, with a visible peak for correctly specified targets

## Limitations

- Screening plots are qualitative visual diagnostics only — they do not produce peak areas or quality metrics and cannot substitute for full integration
- Visibility of a target peak still depends on correct target-list annotation; a missing peak may reflect wrong m/z or RT windows rather than true absence
- Package installation can require raising R's connection timeout (options(timeout = "300")) because the package bundles example data

## Evidence

- [readme] Make sure `R` (**version >= 4.4.0**) is installed on your computer:: [evidence span withheld — anchor status: external_doc]
- [readme] Read the vignettes for a tutorial on how to use TARDIS, available online at https://pablovgd.github.io/TARDIS/: [evidence span withheld — anchor status: external_doc]
