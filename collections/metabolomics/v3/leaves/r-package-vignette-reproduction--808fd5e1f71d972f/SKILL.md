---
name: r-package-vignette-reproduction
description: Use when validating an R package's documented behavior by executing its
  vignette workflow — specifically, confirming that TARDIS screening-mode EIC diagnostic
  plots (five EICs per image for QC runs) appear in the output folder before committing
  to full peak detection.
when_to_use_negative:
- The goal is full quantitative peak integration — screening_mode = TRUE is a pre-detection
  visual check, not the production integration run
- Input data are untargeted/preprocessed feature tables; this skill targets raw mzML
  files in a targeted LC-MS workflow (xcms/MZmine/OpenMS-style untargeted preprocessing
  is a different scope)
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3272
tools:
- name: TARDIS
  role: R package whose screening_mode = TRUE option in tardisPeaks() generates the
    QC-run EIC diagnostic plots
  repo: https://github.com/UGent-LIMET/TARDIS
- name: R
  role: Execution environment (>= 4.4.0, BiocManager 3.20) in which the vignette workflow
    is run
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/acs.analchem.5c00567
    title: Automated Integration and Quality Assessment of Chromatographic Peaks in
      LC–MS-Based Metabolomics and Lipidomics Using TARDIS
  source_content_hash: sha256:5a4d0dfb0064b38003b9a4c7dd970e3f5dc3907bdba1acc57020c0562e96215d
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
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3272
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# r-package-vignette-reproduction

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Reproduces the screening-mode diagnostic workflow from the TARDIS R package vignette: run targeted peak integration with screening_mode = TRUE on the bundled example mzML files and verify that extracted ion chromatogram (EIC) plots are generated in the output folder exactly as shown in the vignette.

## When to use

Use when validating an R package's documented behavior by executing its vignette workflow — specifically, confirming that TARDIS screening-mode EIC diagnostic plots (five EICs per image for QC runs) appear in the output folder before committing to full peak detection.

## When NOT to use

- The goal is full quantitative peak integration — screening_mode = TRUE is a pre-detection visual check, not the production integration run
- Input data are untargeted/preprocessed feature tables; this skill targets raw mzML files in a targeted LC-MS workflow (xcms/MZmine/OpenMS-style untargeted preprocessing is a different scope)

## Inputs

- Example mzML LC-MS run files bundled with the TARDIS R package
- Target list (m/z and retention time windows) as used by tardisPeaks()
- R environment >= 4.4.0 with BiocManager 3.20 and devtools

## Outputs

- Diagnostic EIC plot images (five EICs per image) for QC runs, saved in the output folder
- Confirmation that target peaks are visible in expected m/z/retention-time windows (visual inspection)

## How to apply

Load the example mzML files bundled with the TARDIS package (installed from github.com/UGent-LIMET/TARDIS via BiocManager; note the README's timeout workaround `options(timeout = "300")` since the package ships example data). Load with library(TARDIS), then call tardisPeaks() with screening_mode = TRUE so that QC runs are automatically selected for screening rather than running full targeted peak integration. TARDIS writes diagnostic EIC plot images — five extracted ion chromatograms per image — for the selected QC runs into the designated output folder. Correctness is judged by confirming the EIC images exist in that folder and visually match the corresponding figures reproduced in the TARDIS vignette (quick_start, available at https://pablovgd.github.io/TARDIS/), thereby confirming target peaks are visible within their expected m/z and retention time windows before committing to full peak detection.

## Related tools

- **TARDIS** (R package whose screening_mode = TRUE option in tardisPeaks() generates the QC-run EIC diagnostic plots) — https://github.com/UGent-LIMET/TARDIS
- **R** (Execution environment (>= 4.4.0, BiocManager 3.20) in which the vignette workflow is run)

## Examples

```
library(TARDIS); results <- tardisPeaks(mzml_files, target_list, screening_mode = TRUE)
```

## Evaluation signals

- EIC plot image files exist in the designated output folder after the tardisPeaks(screening_mode = TRUE) call
- Each diagnostic image contains five extracted ion chromatograms, per the vignette specification
- Generated EIC images visually match the corresponding figures shown in the TARDIS vignette
- Screening was restricted to automatically selected QC runs, not all runs in the batch
- Targets appear within their expected m/z and retention-time windows in the inspected EICs

## Limitations

- Package download can hit connection timeouts because the package contains example data; increase R's timeout setting (e.g., options(timeout = "300"))
- Windows users additionally need Rtools; Mac users need Xcode and a GNU Fortran compiler
- Screening is a visual/manual confirmation step — visibility of target peaks in EICs does not guarantee integration quality metrics (AUC, S/N, bell-curve similarity) will pass later filtering

## Evidence

- [readme] readme-install: [evidence span withheld — anchor status: external_doc]
- [readme] readme-vignettes: [evidence span withheld — anchor status: external_doc]
