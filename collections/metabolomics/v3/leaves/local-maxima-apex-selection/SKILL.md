---
name: local-maxima-apex-selection
description: Use when you must define integration limits for targeted LC–MS chromatographic
  peaks from RT/intensity vectors of extracted ion chromatograms, given a target list
  with expected retention times.
when_to_use_negative:
- The analysis goal is untargeted peak finding across the full run — TARDIS is a targeted
  integration tool, unlike untargeted preprocessors such as xcms, MZmine or OpenMS
- Peaks are already integrated and only a feature table exists (no raw RT/intensity
  vectors available for apex/border detection)
- The apex must be a fixed exact retention-time window rather than data-driven local-minimum
  borders, e.g., strict vendor-defined integration regions
edam_topics:
- http://edamontology.org/topic_3652
- http://edamontology.org/topic_3383
tools:
- name: TARDIS
  role: Open-source R package implementing Savitzky-Golay smoothing, local-maxima
    apex selection at the RT nearest the target, local-minimum border detection, and
    the five quality metrics
  repo: https://github.com/UGent-LIMET/TARDIS
- name: R
  role: Runtime environment (version >= 4.4.0; BiocManager 3.20 for installation)
- name: signal
  role: Provides sgolayfilt() for the Savitzky-Golay smoothing step that precedes
    apex and border detection
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/acs.analchem.5c00567
    title: Automated Integration and Quality Assessment of Chromatographic Peaks in
      LC–MS-Based Metabolomics and Lipidomics Using TARDIS
  source_content_hash: sha256:d728655f1d13d5b3098896dd3683f84c23911fa9dfe8bbc4e8d9de843e0e4327
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
  - signal
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3652
  - http://edamontology.org/topic_3383
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# local-maxima-apex-selection

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

TARDIS-style targeted apex and border picking for chromatographic peaks: smooth the intensity vector with a Savitzky-Golay filter, select the apex as the local intensity maximum nearest the target retention time, and set integration borders at the flanking local minima before trapezoidal AUC integration.

## When to use

Use when you must define integration limits for targeted LC–MS chromatographic peaks from RT/intensity vectors of extracted ion chromatograms, given a target list with expected retention times.

## When NOT to use

- The analysis goal is untargeted peak finding across the full run — TARDIS is a targeted integration tool, unlike untargeted preprocessors such as xcms, MZmine or OpenMS
- Peaks are already integrated and only a feature table exists (no raw RT/intensity vectors available for apex/border detection)
- The apex must be a fixed exact retention-time window rather than data-driven local-minimum borders, e.g., strict vendor-defined integration regions

## Inputs

- Target/retention-time list (target name plus expected RT per analyte)
- Ion chromatogram RT and intensity vectors extracted from mzML LC–MS runs (e.g., vignette mzML files shipped with TARDIS)
- R session with TARDIS and the signal package installed

## Outputs

- Peak apex RT and left/right border RTs (flanking local minima) per target per run
- AUC per target per run (data.frame inside the results list)
- QC-run feature table tibble with average per-target metrics (AUC, Max. Int., custom SNR, bell-curve similarity, points over the peak)
- EIC diagnostic plots saved to the output folder

## How to apply

Extract the ion chromatogram (EIC) for each target from the mzML LC–MS runs and smooth its intensity vector with a Savitzky-Golay filter (sgolayfilt() from the R signal package), which TARDIS uses specifically to stabilize border detection. Select the peak apex as the local intensity maximum nearest the target retention time rather than the global maximum, so co-eluting interference does not steal the apex. Define the peak borders at the local minima immediately flanking the selected apex of the smoothed chromatogram; these bounds delimit the integration window. Integrate the bounded peak with trapezoidal AUC and compute the five quality metrics (area under the curve, maximum intensity, points over the peak, custom signal-to-noise ratio, and bell-curve similarity) per target per run. Assemble results as a list containing a data.frame of AUC per target per run plus a tibble feature table of average metrics per target in QC runs. Run under R >= 4.4.0 with BiocManager 3.20, installing TARDIS from GitHub.

## Related tools

- **TARDIS** (Open-source R package implementing Savitzky-Golay smoothing, local-maxima apex selection at the RT nearest the target, local-minimum border detection, and the five quality metrics) — https://github.com/UGent-LIMET/TARDIS
- **R** (Runtime environment (version >= 4.4.0; BiocManager 3.20 for installation))
- **signal** (Provides sgolayfilt() for the Savitzky-Golay smoothing step that precedes apex and border detection)

## Examples

```
BiocManager::install("pablovgd/TARDIS", build_vignettes = FALSE, dependencies=TRUE); library(TARDIS)
```

## Evaluation signals

- The selected apex is the local intensity maximum nearest the target retention time, not necessarily the global maximum of the chromatogram
- Peak borders fall exactly at the flanking local minima of the Savitzky-Golay-smoothed chromatogram on both sides of the apex
- AUC values equal trapezoidal integration between the detected borders, and the five metrics (AUC, Max. Int., custom SNR, bell-curve similarity, points over the peak) are present for every target in every run
- The results object is a list containing a data.frame of AUC per target per run plus a tibble feature table of average metrics per target in QC runs, matching TARDIS's reported structure
- Per-target AUC and QC feature-table values are cross-checkable against the published vignette outputs; residual peak-shape discrepancies are routed to expert review

## Limitations

- Border detection depends on local minima of the smoothed signal, so heavily noisy or shoulder-dominated chromatograms may shift borders; discrepancies require expert review
- No version number for the TARDIS R package is stated in the article; the README only pins R >= 4.4.0 and BiocManager 3.20
- Installation with example data often hits R connection timeouts, requiring options(timeout = '300')
- The article's vendor-software comparison is described only generically as 'state-of-the-art vendor software', so expected border agreement with specific tools cannot be pinned

## Evidence

- [other] TARDIS smooths the chromatographic intensity vector with a Savitzky-Golay filter (sgolayfilt() from the signal package) as part of targeted peak border detection.: [evidence span withheld — anchor status: not_found]
- [readme] Make sure `R` (**version >= 4.4.0**) is installed on your computer:: [evidence span withheld — anchor status: external_doc]
