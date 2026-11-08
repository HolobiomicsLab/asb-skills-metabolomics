---
name: savitzky-golay-chromatogram-smoothing
description: Use when targeted LC–MS ion chromatograms (RT and intensity vectors)
  are noisy and peak borders must be located reliably before integration, as in TARDIS's
  targeted peak border detection workflow.
when_to_use_negative:
- The goal is untargeted feature finding across full profiles — TARDIS and this smoothing/border-detection
  step are for targeted peak extraction, not untargeted peak searching (xcms, MZmine,
  OpenMS).
- Peaks are already integrated and only downstream statistics or feature-table manipulation
  is needed — no chromatogram smoothing is required.
- The input is already a QC feature table of per-target average metrics rather than
  raw chromatographic intensity vectors.
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: TARDIS
  role: Open-source R package whose targeted peak border detection applies the Savitzky-Golay
    smoothing before apex selection, border definition at flanking local minima, trapezoidal
    AUC integration, and quality-metric computation
  repo: https://github.com/UGent-LIMET/TARDIS
- name: R
  role: Runtime environment (version >= 4.4.0) in which TARDIS and the Savitzky-Golay
    smoothing are executed
- name: signal
  role: R package supplying the sgolayfilt() Savitzky-Golay filter function applied
    to the chromatographic intensity vector
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/acs.analchem.5c00567
    title: Automated Integration and Quality Assessment of Chromatographic Peaks in
      LC–MS-Based Metabolomics and Lipidomics Using TARDIS
  source_content_hash: sha256:5ec1c31215016b264663214127c8e7f1ff0b751fce2cd7e6aad65484817efe01
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
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# savitzky-golay-chromatogram-smoothing

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Savitzky-Golay smoothing of chromatographic intensity vectors, as used inside the TARDIS R package (via sgolayfilt() from the signal package) to stabilize targeted LC–MS ion chromatograms before apex selection, peak border definition, and trapezoidal AUC integration.

## When to use

Use when targeted LC–MS ion chromatograms (RT and intensity vectors) are noisy and peak borders must be located reliably before integration, as in TARDIS's targeted peak border detection workflow.

## When NOT to use

- The goal is untargeted feature finding across full profiles — TARDIS and this smoothing/border-detection step are for targeted peak extraction, not untargeted peak searching (xcms, MZmine, OpenMS).
- Peaks are already integrated and only downstream statistics or feature-table manipulation is needed — no chromatogram smoothing is required.
- The input is already a QC feature table of per-target average metrics rather than raw chromatographic intensity vectors.

## Inputs

- Extracted-ion chromatograms (RT vector + intensity vector) per target per run, extracted from raw LC–MS mzML files
- Target/retention-time list defining the metabolites and lipids to integrate
- R >= 4.4.0 environment with the TARDIS package (installed via BiocManager) and the signal package for sgolayfilt()

## Outputs

- Smoothed chromatogram intensity vectors per target per run
- Per-target peak apex and peak borders (flanking local minima)
- Per-target integrated peak area (trapezoidal AUC) and five quality metrics (Max. Int., custom signal-to-noise ratio, bell-curve similarity, points-over-the-peak)
- TARDIS results object: a list containing a data.frame of AUC of each target in each run and a tibble feature table of average metrics per target in QC runs

## How to apply

Extract the extracted-ion chromatogram (EIC) for each target from the raw mzML run and pass the intensity vector (paired with RT) to a Savitzky-Golay filter, e.g. signal::sgolayfilt(), as TARDIS does in its peak-border detection. On the smoothed chromatogram, select the apex as the local intensity maximum nearest the target retention time, then define the peak borders at the flanking local minima. Integrate the bounded peak by trapezoidal AUC and compute the five TARDIS quality metrics (AUC, maximum intensity, points-over-the-peak, custom signal-to-noise ratio, and bell-curve similarity). The rationale is that Savitzky-Golay filtering preserves peak shape and height while suppressing noise, which makes local-minimum border detection reproducible across the 1786 benchmarked runs; TARDIS's smoothed-vector integration was benchmarked as comparable to vendor software. Any residual peak-shape discrepancies between TARDIS and reference outputs are escalated to expert review rather than silently re-integrated.

## Related tools

- **TARDIS** (Open-source R package whose targeted peak border detection applies the Savitzky-Golay smoothing before apex selection, border definition at flanking local minima, trapezoidal AUC integration, and quality-metric computation) — https://github.com/UGent-LIMET/TARDIS
- **R** (Runtime environment (version >= 4.4.0) in which TARDIS and the Savitzky-Golay smoothing are executed)
- **signal** (R package supplying the sgolayfilt() Savitzky-Golay filter function applied to the chromatographic intensity vector)

## Examples

```
library(TARDIS); library(signal); smoothed_ic <- sgolayfilt(ic$intensity)
```

## Evaluation signals

- Peak apex on the smoothed chromatogram must be the local intensity maximum nearest the target retention time
- Peak borders must fall at the flanking local minima of the smoothed chromatogram for each target
- The results object must match TARDIS's reported structure: a list with a data.frame of AUC of each target in each run plus a tibble of average metrics per target in QC runs
- AUC values should be cross-checkable against the vignette's published outputs on the shipped mzML example runs
- Integrated peak areas should be reliable and comparable to state-of-the-art vendor software, per the article's benchmark across three biofluids and two LC–MS instruments

## Limitations

- The article reports no version number for the TARDIS R package, and smoothing parameters (e.g., Savitzky-Golay window length and polynomial order) are not stated in the available context and must be taken from the TARDIS source.
- Benchmark comparison targets are described only generically as 'state-of-the-art vendor software' without names or versions.
- Residual peak-shape discrepancies between TARDIS and reference outputs are routed to expert review rather than resolved automatically.
- The paper reports no explicit limitations of TARDIS in its discussion section, so edge cases (e.g., co-elution or absent peaks) are not documented in the provided context.

## Evidence

- [other] TARDIS smooths the chromatographic intensity vector with a Savitzky-Golay filter (sgolayfilt() from the signal package) as part of targeted peak border detection.: [evidence span withheld — anchor status: not_found]
