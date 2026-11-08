---
name: chromatographic-quality-metric-calculation
description: Use when targeted LC–MS peak integration (e.g., TARDIS on raw metabolomics/lipidomics
  runs) has produced per-target peak tables and you need quantitative quality metrics
  to filter poor-quality peaks or validate QC-run clustering.
when_to_use_negative:
- Data are untargeted — the article positions TARDIS for targeted peak extraction,
  noting that 'targeted peak extraction outperformed untargeted peak searching'; use
  untargeted tools like xcms, MZmine, or OpenMS instead.
- Input is already a processed feature/peak table — quality metrics must be computed
  during integration from raw chromatographic data.
edam_operation: http://edamontology.org/operation_3219
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: TARDIS
  role: R package that performs targeted peak integration and computes the quality
    metrics (AUC, maximum intensity, points over the peak, custom SNR, bell-curve
    similarity)
  repo: https://github.com/UGent-LIMET/TARDIS
- name: R
  role: Runtime environment for TARDIS; used to inspect the results list, extract
    AUC data.frames, and run PCA on the metric tables
  repo: https://cloud.r-project.org/index.html
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1021/acs.analchem.5c00567
    title: Automated Integration and Quality Assessment of Chromatographic Peaks in
      LC–MS-Based Metabolomics and Lipidomics Using TARDIS
  source_content_hash: sha256:0a8e4ab01a4cc57e5c60bb2b7b40cff18d7d2ced7718d406142bed50ca0a4fcb
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
  edam_operation: http://edamontology.org/operation_3219
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# chromatographic-quality-metric-calculation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Compute TARDIS's peak-quality metrics (area under the curve, maximum intensity, points over the peak, custom signal-to-noise ratio, and bell-curve similarity) for targeted LC–MS chromatographic peaks to assess integration quality before downstream statistics.

## When to use

Use when targeted LC–MS peak integration (e.g., TARDIS on raw metabolomics/lipidomics runs) has produced per-target peak tables and you need quantitative quality metrics to filter poor-quality peaks or validate QC-run clustering.

## When NOT to use

- Data are untargeted — the article positions TARDIS for targeted peak extraction, noting that 'targeted peak extraction outperformed untargeted peak searching'; use untargeted tools like xcms, MZmine, or OpenMS instead.
- Input is already a processed feature/peak table — quality metrics must be computed during integration from raw chromatographic data.

## Inputs

- Raw targeted LC–MS metabolomics/lipidomics run files (e.g., from studies like MTBLS8735)
- Target list of chromatographic peaks (e.g., the target list distributed with the TARDIS case_study vignette)

## Outputs

- data.frame of AUC (and quality metrics) per target per run from the TARDIS results list
- tibble feature table with average quality metrics for each target in QC runs
- EIC diagnostic plots saved in the output folder for inspection

## How to apply

Run TARDIS targeted peak integration over the raw LC–MS files with a target list so that, per target per run, the package computes both established metrics (area under the curve, maximum intensity, points over the peak) and recently developed ones (custom signal-to-noise ratio and bell-curve similarity). Extract the results object, which is a list containing a data.frame with the AUC of each target in each run plus a tibble holding a feature table of average metrics for each target in the QC runs. Use these metrics to evaluate peak quality, optionally enhancing peak filtering by quality with machine learning classification as explored in the paper. Validate by confirming that QC runs show consistent, high-quality metric values (e.g., tightly clustered in a PCA of integrated AUCs) while biological samples retain expected variance.

## Related tools

- **TARDIS** (R package that performs targeted peak integration and computes the quality metrics (AUC, maximum intensity, points over the peak, custom SNR, bell-curve similarity)) — https://github.com/UGent-LIMET/TARDIS
- **R** (Runtime environment for TARDIS; used to inspect the results list, extract AUC data.frames, and run PCA on the metric tables) — https://cloud.r-project.org/index.html

## Examples

```
BiocManager::install("pablovgd/TARDIS", build_vignettes = FALSE, dependencies=TRUE); library(TARDIS)
```

## Evaluation signals

- The TARDIS results object is a list containing a data.frame with an AUC value for each target in each run, plus a tibble of average metrics per target in QC runs.
- Exploratory PCA of integrated target AUCs shows QC runs tightly clustered while sample runs show variance, as expected for high-quality data sets.
- All five metrics (AUC, maximum intensity, points over the peak, custom SNR, bell-curve similarity) are populated for integrated targets.
- Peak integration results are reliable and comparable to state-of-the-art vendor software.
- EIC diagnostic plots saved in the output folder can be visually inspected to confirm correct peak integration.

## Limitations

- No version number for the TARDIS R package is stated in the discussion; rely on the README requirement of R >= 4.4.0 and BiocManager 3.20.
- Comparison to vendor software is only described generically ('state-of-the-art vendor software') and the specific software is never named in that context.
- Quality metrics must come from targeted integration of raw data; they are not applicable to already-processed peak tables.

## Evidence

- [abstract] Established and recent quality metrics included in TARDIS: "Both established (area under the curve, maximum intensity and points over the peak) and recently developed (custom signal-to-noise ratio and bell-curve similarity) quality metrics were included to"
- [results] Results object structure: "The `results` object is a `list` that contains a `data.frame` with the AUC of each target in each run"
- [results] QC feature table: "a `tibble` that contains a feature table with the average metrics for each target in the QC runs."
- [intro] Targeted over untargeted rationale: "targeted peak extraction outperformed untargeted peak searching"
- [readme] Installation requirements from README: [evidence span withheld — anchor status: external_doc]
