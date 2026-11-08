---
name: peak-quality-metric-computation
description: Use when integrating targeted LC–MS peaks from RT/intensity ion chromatograms
  and you need per-target quality metrics (AUC, Max. Int., custom S/N, bell-curve
  similarity, points over the peak) to filter or QC peaks across large run collections.
when_to_use_negative:
- The analysis goal is untargeted peak detection — TARDIS is built for targeted peak
  extraction, where untargeted peak searching was reported to be outperformed by targeted
  extraction
- Only a finished feature table is available and raw chromatograms (RT/intensity vectors)
  no longer exist, so the five metrics cannot be recomputed from peaks
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: TARDIS
  role: R package whose targeted peak-border detection, integration, and five-metric
    quality assessment are implemented by this skill
  repo: https://github.com/UGent-LIMET/TARDIS
- name: R
  role: Runtime environment for TARDIS (requires R >= 4.4.0; install via BiocManager)
  repo: https://cloud.r-project.org/index.html
- name: signal
  role: Provides sgolayfilt() for Savitzky-Golay smoothing of the chromatogram prior
    to border detection
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/acs.analchem.5c00567
    title: Automated Integration and Quality Assessment of Chromatographic Peaks in
      LC–MS-Based Metabolomics and Lipidomics Using TARDIS
  source_content_hash: sha256:c416a262ae08afa1df491a9cac6099e6380a8a054a00ec9cd3f02cb681f5bfa5
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
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# peak-quality-metric-computation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Computes the five TARDIS quality metrics — trapezoidal AUC, Max. Int., a custom signal-to-noise ratio, bell-curve similarity, and points-over-the-peak — for every targeted chromatographic peak in every LC–MS run, enabling automated peak quality assessment in metabolomics and lipidomics.

## When to use

Use when integrating targeted LC–MS peaks from RT/intensity ion chromatograms and you need per-target quality metrics (AUC, Max. Int., custom S/N, bell-curve similarity, points over the peak) to filter or QC peaks across large run collections.

## When NOT to use

- The analysis goal is untargeted peak detection — TARDIS is built for targeted peak extraction, where untargeted peak searching was reported to be outperformed by targeted extraction
- Only a finished feature table is available and raw chromatograms (RT/intensity vectors) no longer exist, so the five metrics cannot be recomputed from peaks

## Inputs

- Ion chromatograms (RT and intensity vectors) extracted per target from mzML LC–MS runs
- Target list with target retention times (e.g., the target/retention-time list shipped with the TARDIS vignette)
- Identification of which runs are QC runs (for the QC-average feature table)

## Outputs

- A results list containing a data.frame with the AUC of each target in each run
- A tibble with a feature table of average metrics for each target in the QC runs
- Five-CSV export layout (AUC per target per run, per-run metrics, QC-average feature table)
- EIC diagnostic plots saved in the output folder for inspection

## How to apply

First detect peak borders on the chromatogram: smooth the intensity vector with a Savitzky-Golay filter (sgolayfilt() from the signal package), select the apex as the local intensity maximum nearest the target retention time, and set borders at the flanking local minima. Then integrate the peak with trapezoidal AUC and compute the metric battery — Max. Int., custom signal-to-noise ratio, bell-curve similarity, and points-over-the-peak — for every target in every run. Assemble results into TARDIS's five-CSV layout: per-target-per-run AUC tables, per-run metrics, and a QC-average feature table. Rationale: established metrics (AUC, maximum intensity, points over the peak) capture peak size and sampling density, while the recently developed custom S/N and bell-curve similarity capture shape quality, together enabling automated peak filtering that was validated against state-of-the-art vendor software across 1786 runs, three biofluids, and two LC–MS instruments.

## Related tools

- **TARDIS** (R package whose targeted peak-border detection, integration, and five-metric quality assessment are implemented by this skill) — https://github.com/UGent-LIMET/TARDIS
- **R** (Runtime environment for TARDIS (requires R >= 4.4.0; install via BiocManager)) — https://cloud.r-project.org/index.html
- **signal** (Provides sgolayfilt() for Savitzky-Golay smoothing of the chromatogram prior to border detection)

## Examples

```
library(TARDIS); if (!require("BiocManager")) install.packages("BiocManager"); BiocManager::install("pablovgd/TARDIS", build_vignettes = FALSE, dependencies = TRUE)
```

## Evaluation signals

- The results object is a list containing a data.frame with the AUC of each target in each run (structural check)
- The results list also contains a tibble with a feature table of average metrics for each target in the QC runs
- All five metrics (AUC, Max. Int., custom S/N, bell-curve similarity, points over the peak) are populated for every target in every run
- Computed AUC and metrics can be cross-checked against the TARDIS vignette's published outputs; residual peak-shape discrepancies go to expert review
- Benchmark sanity check: peak integration results should be reliable and comparable to state-of-the-art vendor software on the same runs

## Limitations

- Residual peak-shape discrepancies between TARDIS and vendor software require expert review
- No version number for the TARDIS R package is stated in the article discussion
- The comparison vendor software is described only generically ('state-of-the-art vendor software') and is never named or versioned
- Package installation contains example data, so slow connections can be an issue — increase the R timeout option (e.g., options(timeout = '300'))

## Evidence
