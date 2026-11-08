---
name: trapezoidal-auc-integration
description: Use when raw targeted LC–MS metabolomics/lipidomics runs must be integrated
  per target from RT and intensity vectors, producing trapezoidal AUC plus quality
  metrics for every target in every run.
when_to_use_negative:
- The goal is untargeted peak detection/feature finding — the article positions xcms,
  MZmine and OpenMS for untargeted preprocessing, whereas TARDIS is targeted peak
  extraction/integration
- Input is already a preprocessed feature table with no raw chromatographic (RT, intensity)
  data — there are no intensity vectors to smooth, bound, and integrate
- Vendor-software-style interactive manual integration is required; TARDIS is an automated,
  scripted R workflow
edam_operation: http://edamontology.org/operation_0279
edam_topics:
- http://edamontology.org/topic_3370
- http://edamontology.org/topic_0092
tools:
- name: TARDIS
  role: R package that performs targeted border detection, trapezoidal AUC integration,
    and quality-metric computation on LC–MS ion chromatograms
  repo: https://github.com/UGent-LIMET/TARDIS
- name: R
  role: Runtime environment; TARDIS requires R >= 4.4.0 (and Rtools/Xcode toolchains
    on Windows/Mac)
  repo: https://cloud.r-project.org/index.html
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/acs.analchem.5c00567
    title: Automated Integration and Quality Assessment of Chromatographic Peaks in
      LC–MS-Based Metabolomics and Lipidomics Using TARDIS
  source_content_hash: sha256:308e3c3f553aa07277c6ee0829ddd745900d7a33fc82636acd1636feee9543b3
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
  edam_operation: http://edamontology.org/operation_0279
  edam_topics:
  - http://edamontology.org/topic_3370
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# trapezoidal-auc-integration

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Targeted integration of LC–MS chromatographic peaks via trapezoidal AUC, as implemented in the TARDIS R package: ion chromatograms are Savitzky-Golay smoothed, borders are set at flanking local minima around the apex nearest the target RT, and AUC plus four quality metrics are computed per target per run.

## When to use

Use when raw targeted LC–MS metabolomics/lipidomics runs must be integrated per target from RT and intensity vectors, producing trapezoidal AUC plus quality metrics for every target in every run.

## When NOT to use

- The goal is untargeted peak detection/feature finding — the article positions xcms, MZmine and OpenMS for untargeted preprocessing, whereas TARDIS is targeted peak extraction/integration
- Input is already a preprocessed feature table with no raw chromatographic (RT, intensity) data — there are no intensity vectors to smooth, bound, and integrate
- Vendor-software-style interactive manual integration is required; TARDIS is an automated, scripted R workflow

## Inputs

- mzML LC–MS raw runs (e.g., the vignette LC-MS runs shipped with TARDIS)
- Target/retention-time list defining the compounds to integrate
- RT and intensity vectors per extracted ion chromatogram

## Outputs

- data.frame of trapezoidal AUC per target per run (inside a results list)
- tibble feature table of average metrics per target in QC runs
- Per-target metric tables exported in TARDIS's five-CSV layout
- EIC diagnostic plots saved in the output folder for inspection

## How to apply

For each target, extract the ion chromatogram (EIC) from the raw mzML run and smooth the intensity vector with a Savitzky-Golay filter (sgolayfilt() from the R signal package). Select the peak apex as the local intensity maximum nearest the target retention time, then define the peak borders at the flanking local minima of the smoothed chromatogram. Integrate the bounded peak by trapezoidal AUC and, in TARDIS, also compute Max. Int., a custom signal-to-noise ratio, bell-curve similarity, and points-over-the-peak. Assemble results as TARDIS does: a list containing a data.frame with the AUC of each target in each run, plus a tibble feature table of average metrics per target in QC runs, exported in the five-CSV layout.

## Related tools

- **TARDIS** (R package that performs targeted border detection, trapezoidal AUC integration, and quality-metric computation on LC–MS ion chromatograms) — https://github.com/UGent-LIMET/TARDIS
- **R** (Runtime environment; TARDIS requires R >= 4.4.0 (and Rtools/Xcode toolchains on Windows/Mac)) — https://cloud.r-project.org/index.html

## Examples

```
BiocManager::install("pablovgd/TARDIS", build_vignettes = TRUE, dependencies=TRUE); library(TARDIS)
```

## Evaluation signals

- The results object is a list containing a data.frame with the AUC of each target in each run, matching TARDIS's reported structure
- The results list contains a tibble with a feature table of average metrics for each target in the QC runs
- AUC values are cross-checkable against the vignette's published outputs for the same mzML inputs and target list
- Peak borders fall on the flanking local minima of the Savitzky-Golay-smoothed chromatogram around the apex nearest the target RT
- All five metrics (AUC, Max. Int., custom signal-to-noise ratio, bell-curve similarity, points-over-the-peak) are populated for every target in every run

## Limitations

- Residual peak-shape discrepancies versus expected targets go to expert review rather than being auto-resolved
- The comparison in the paper is against generic 'state-of-the-art vendor software' that is never named or versioned
- No version number for the TARDIS R package is stated in the article; the README specifies R >= 4.4.0 and BiocManager 3.20 for installation

## Evidence
