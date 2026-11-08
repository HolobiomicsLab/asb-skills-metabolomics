---
name: chromatographic-eic-visualization
description: Use when targeted LC–MS metabolomics/lipidomics data (mzML raw runs)
  must be sanity-checked before peak integration — e.g., verifying that target peaks
  are visible within expected m/z and RT windows in QC runs prior to running full
  TARDIS integration.
when_to_use_negative:
- When full quantitative peak integration results (AUC per target per run) are already
  needed — screening mode only produces EIC visuals and QC-run feature summaries,
  not final peak areas
- When input data are already processed feature tables rather than raw mzML chromatographic
  runs
- For untargeted peak detection workflows (e.g., xcms, MZmine, OpenMS preprocessing),
  which TARDIS screening is not designed for
edam_operation: http://edamontology.org/operation_3203
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: TARDIS
  role: R package providing tardisPeaks(screening_mode = TRUE) to extract EICs for
    QC runs and render the diagnostic plot images
  repo: https://github.com/UGent-LIMET/TARDIS
- name: R
  role: Runtime environment for executing TARDIS screening mode and viewing the saved
    EIC diagnostic images
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/acs.analchem.5c00567
    title: Automated Integration and Quality Assessment of Chromatographic Peaks in
      LC–MS-Based Metabolomics and Lipidomics Using TARDIS
  source_content_hash: sha256:777838fea60bc65405ff9cf012567138fa6c27607c5e744020af81c116f1292b
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
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# chromatographic-eic-visualization

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A screening-mode QC procedure in the TARDIS R package that runs targeted peak integration with screening_mode = TRUE on QC runs and saves extracted ion chromatogram (EIC) diagnostic plots (five EICs per image) so the analyst can visually confirm target peaks are present in their expected m/z and retention time windows before committing to full peak detection.

## When to use

Use when targeted LC–MS metabolomics/lipidomics data (mzML raw runs) must be sanity-checked before peak integration — e.g., verifying that target peaks are visible within expected m/z and RT windows in QC runs prior to running full TARDIS integration.

## When NOT to use

- When full quantitative peak integration results (AUC per target per run) are already needed — screening mode only produces EIC visuals and QC-run feature summaries, not final peak areas
- When input data are already processed feature tables rather than raw mzML chromatographic runs
- For untargeted peak detection workflows (e.g., xcms, MZmine, OpenMS preprocessing), which TARDIS screening is not designed for

## Inputs

- Raw LC–MS runs in mzML format (targeted assay, including QC runs)
- Target list with expected m/z and retention time windows
- Designated output folder for diagnostic plot images

## Outputs

- Diagnostic EIC plot images (five extracted ion chromatograms per image) saved in the output folder for the selected QC runs
- Visual confirmation of target peak visibility within expected m/z and RT windows

## How to apply

Load the example or study mzML files bundled with TARDIS (install via BiocManager from the UGent-LIMET/TARDIS GitHub repo; requires R >= 4.4.0 and BiocManager 3.20). Invoke tardisPeaks() with screening_mode = TRUE, which automatically selects the QC runs for screening and performs targeted extraction only, skipping full peak detection. TARDIS then renders diagnostic EIC images for the selected QC runs, grouped five EICs per image, and writes them into the designated output folder. Inspect each image to confirm each target's chromatographic peak appears inside the expected m/z and retention time window and shows acceptable shape/intensity before re-running full targeted peak integration; the corresponding vignette figures (quick_start vignette) serve as the visual reference for what correct output looks like.

## Related tools

- **TARDIS** (R package providing tardisPeaks(screening_mode = TRUE) to extract EICs for QC runs and render the diagnostic plot images) — https://github.com/UGent-LIMET/TARDIS
- **R** (Runtime environment for executing TARDIS screening mode and viewing the saved EIC diagnostic images)

## Examples

```
results <- TARDIS::tardisPeaks(files = mzml_files, targets = target_list, output_folder = "eic_screening", screening_mode = TRUE)
```

## Evaluation signals

- EIC image files exist in the designated output folder after running tardisPeaks(screening_mode = TRUE)
- Each generated image contains exactly five EIC traces as specified by the screening workflow
- Plots correspond only to QC runs selected automatically by screening mode
- Visual comparison against the corresponding figures in the TARDIS vignette matches (same targets, m/z windows, RT windows visible)
- Target peaks are visibly located within their expected m/z and retention time windows in the rendered chromatograms

## Limitations

- Screening mode is restricted to QC runs and produces diagnostic EICs only; it does not replace full peak integration
- Judgment of target visibility is visual/manual — no automated pass/fail threshold on the EIC plots is described
- R >= 4.4.0, BiocManager 3.20, Rtools (Windows) or Xcode + GNU Fortran (Mac) are required; package installation can hit connection timeouts due to bundled example data

## Evidence

- [results] The resulting EICs are again saved in the output folder and can be inspected:: "The resulting EICs are again saved in the output folder and can be inspected:"
- [abstract] We therefore present TARDIS, an open-source R package for the analysis of targeted LC–MS metabolomics and lipidomics data.: "We therefore present TARDIS, an open-source R package for the analysis of targeted LC–MS metabolomics and lipidomics data."
- [methods] Quick start for targeted peak integration of LC-MS data using TARDIS: "Quick start for targeted peak integration of LC-MS data using TARDIS"
- [readme] Read the vignettes for a tutorial on how to use TARDIS: [evidence span withheld — anchor status: external_doc]
