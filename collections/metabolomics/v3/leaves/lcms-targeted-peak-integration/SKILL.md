---
name: lcms-targeted-peak-integration
description: Use when raw targeted LC–MS runs (e.g., mzML) must be integrated per
  target list/retention-time list, producing per-target AUC and quality metrics (AUC,
  Max. Int., SNR, bell-curve similarity, points over the peak) for large cohorts.
when_to_use_negative:
- Untargeted peak detection is needed — TARDIS performs targeted peak extraction and
  is not a substitute for untargeted tools like xcms, MZmine or OpenMS
- Input is already a processed feature table rather than raw chromatographic data
  requiring integration
- Vendor-specific integration is required, e.g., Thermo Scientific TraceFinder or
  Xcalibur, Waters ChromScope, or SCIEX OS-MQ
edam_operation: http://edamontology.org/operation_3215
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0154
tools:
- name: TARDIS
  role: Open-source R package that performs targeted raw data integration in spectrometry
    and peak quality evaluation
  repo: https://github.com/UGent-LIMET/TARDIS
- name: R
  role: Runtime environment (R >= 4.4.0); install TARDIS via BiocManager 3.20 with
    devtools
  repo: https://cloud.r-project.org/index.html
- name: UGent-LIMET/tardis_publication_code
  role: R and python scripts to reproduce the data-analysis and figures presented
    in Vangeenderhuysen et al. 2025
  repo: https://github.com/UGent-LIMET/tardis_publication_code
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/acs.analchem.5c00567
    title: Automated Integration and Quality Assessment of Chromatographic Peaks in
      LC–MS-Based Metabolomics and Lipidomics Using TARDIS
  source_content_hash: sha256:8612d751edc6b4f09b2b643330dd8be6a3d1d9feffdc9a5602ffbcbd4f4eff06
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
  - UGent-LIMET/tardis_publication_code
  edam_operation: http://edamontology.org/operation_3215
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# lcms-targeted-peak-integration

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Automated targeted integration and quality assessment of chromatographic peaks in LC–MS metabolomics/lipidomics using the TARDIS R package, which combines peak integration (trapezoidal AUC) with five quality metrics to enable fast, reliable, reproducible integration in large-scale assays.

## When to use

Use when raw targeted LC–MS runs (e.g., mzML) must be integrated per target list/retention-time list, producing per-target AUC and quality metrics (AUC, Max. Int., SNR, bell-curve similarity, points over the peak) for large cohorts.

## When NOT to use

- Untargeted peak detection is needed — TARDIS performs targeted peak extraction and is not a substitute for untargeted tools like xcms, MZmine or OpenMS
- Input is already a processed feature table rather than raw chromatographic data requiring integration
- Vendor-specific integration is required, e.g., Thermo Scientific TraceFinder or Xcalibur, Waters ChromScope, or SCIEX OS-MQ

## Inputs

- Raw LC–MS runs in mzML format (targeted metabolomics/lipidomics)
- Target list with retention times (target/retention-time list shipped with TARDIS)
- QC run annotations (to compute QC-average feature table)

## Outputs

- List containing a data.frame with the AUC of each target in each run
- Tibble feature table with average metrics for each target in the QC runs
- Five-CSV layout of per-target metric tables (AUC per target per run, per-run metrics)
- EIC diagnostic plots for QC components saved in output folder

## How to apply

Load the vignette mzML LC-MS runs and the target/retention-time list into TARDIS from github.com/UGent-LIMET/TARDIS. For each target, extract the ion chromatogram and smooth the intensity vector with a Savitzky-Golay filter (sgolayfilt() from the R signal package); select the apex as the local intensity maximum nearest the target RT and define borders at the flanking local minima of the smoothed chromatogram. Integrate each peak by trapezoidal AUC and compute the five quality metrics: area under the curve, maximum intensity, custom signal-to-noise ratio, bell-curve similarity, and points over the peak. Export per-target metric tables in TARDIS's five-CSV layout, where the results object is a list containing a data.frame of AUC of each target in each run plus a tibble feature table of average metrics per target in QC runs; optionally launch the GUI via runTardis().

## Related tools

- **TARDIS** (Open-source R package that performs targeted raw data integration in spectrometry and peak quality evaluation) — https://github.com/UGent-LIMET/TARDIS
- **R** (Runtime environment (R >= 4.4.0); install TARDIS via BiocManager 3.20 with devtools) — https://cloud.r-project.org/index.html
- **UGent-LIMET/tardis_publication_code** (R and python scripts to reproduce the data-analysis and figures presented in Vangeenderhuysen et al. 2025) — https://github.com/UGent-LIMET/tardis_publication_code

## Examples

```
BiocManager::install("pablovgd/TARDIS", build_vignettes = FALSE, dependencies=TRUE); library(TARDIS); runTardis()
```

## Evaluation signals

- The results object is a list containing a data.frame with the AUC of each target in each run, matching TARDIS's documented output structure
- A tibble feature table with the average metrics for each target in the QC runs is present and populated
- AUC values are cross-checkable against the vignette's published outputs from the mzML example data
- All five metrics (AUC, Max. Int., custom signal-to-noise ratio, bell-curve similarity, points over the peak) are computed for every target in every run
- Integration results are comparable to state-of-the-art vendor software, as benchmarked across three biofluids and two LC–MS instruments for 1786 runs

## Limitations

- Peak-shape discrepancies between TARDIS and vendor software may require expert review
- Connection timeout issues when installing since the package contains example data (increase R timeout, e.g., options(timeout = "300"))
- Windows users need Rtools and Mac users need Xcode plus a GNU Fortran compiler for installation
- The article describes comparison vendor software only generically and states no version number for the TARDIS R package

## Evidence

- [methods] TARDIS smooths the chromatographic intensity vector with a Savitzky-Golay filter (sgolayfilt() from the signal package) as part of targeted peak border detection.: [evidence span withheld — anchor status: not_found]
