---
name: retention-time-peak-boundary-assessment
description: Use when building an LC-MS metabolite RT library from mzML files of authentic
  standards and you must decide which detected EIC peaks have valid boundaries and
  RTs worth exporting.
when_to_use_negative:
- Input is already a validated feature table or library with assigned RTs and quality
  labels rather than raw mzML standard runs.
- The goal is untargeted feature detection in biological samples — metScribeR is designed
  for processing LC-MS metabolite standards data, not experimental sample peak picking.
- MS/MS-driven identification is required — metScribeR focuses library building on
  MS1 and RT data and does not use or require MS2 for library construction.
edam_operation: http://edamontology.org/operation_3215
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: metScribeR
  role: R/Shiny package providing the Find Peaks, Review Results, and View/Export
    Library tabs for boundary visualization, manual peak labeling, and library export
  repo: https://github.com/ncats/metScribeR
- name: chromatographR
  role: supplies the find_peaks peak-detection routine applied to each adduct's EIC
    within metScribeR
  repo: https://github.com/ethanbass/chromatographR
- name: R
  role: runtime environment in which metScribeR is installed via devtools and the
    Shiny app is launched
  repo: https://github.com/ncats/metScribeR
- name: Shiny
  role: application framework hosting the interactive peak review and export workflow
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/acs.jproteome.5c00548
    title: 'MetScribeR: A Semiautomated Tool for Data Processing of In-House LC-MS
      Metabolite Reference Libraries'
  source_content_hash: sha256:a36aca33b45ab394ad0ec25489e9bb4ae4e541b179d764774b7eca653477d62a
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/ncats/metScribeR
  repo_url: https://github.com/ncats/metScribeR
  tools:
  - R
  - Shiny
  - chromatographR
  - metScribeR
  edam_operation: http://edamontology.org/operation_3215
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 16
---

# retention-time-peak-boundary-assessment

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Assess and assign retention-time peak boundaries for metabolite standard adducts by combining chromatographR-derived peak detection on EICs with metScribeR's automated quality filters and manual Good/Bad/Multimodal review in the Shiny app, so that only trustworthy RTs enter the final library.

## When to use

Use when building an LC-MS metabolite RT library from mzML files of authentic standards and you must decide which detected EIC peaks have valid boundaries and RTs worth exporting.

## When NOT to use

- Input is already a validated feature table or library with assigned RTs and quality labels rather than raw mzML standard runs.
- The goal is untargeted feature detection in biological samples — metScribeR is designed for processing LC-MS metabolite standards data, not experimental sample peak picking.
- MS/MS-driven identification is required — metScribeR focuses library building on MS1 and RT data and does not use or require MS2 for library construction.

## Inputs

- mzML files of LC-MS runs of authentic standards (positive and negative ESI)
- standards_df.csv with common_name, monoisotopic_mass, and pos/neg_mode_mzML_file_path columns
- adduct_df.csv with adduct, change_from_neutral, and mode (POS/NEG) columns
- user-defined background noise level, m/z tolerance, and RT tolerance
- metScribeR storage_object.RDS from a prior session

## Outputs

- Per-peak boundary and RT assignments with quality metrics
- exported_metScribeR_library.csv containing only 'good' adducts with identity, m/z, RT, and quality information
- exported_metScribeR_library_with_metrics.csv including good, bad, and multimodal/indeterminate adducts
- PNG figures of each manually reviewed peak in the Figures directory
- storage_object.RDS capturing app state for resuming

## How to apply

Run peak finding on each adduct's extracted ion chromatogram using metScribeR's chromatographR-derived find_peaks routine with user-tuned density filtering and data smoothing, then apply the automated thresholds: at least 7 data points per peak in a single EIC, width at half maximum intensity greater than 0 s and less than 15 s, no more than 250 peaks per EIC, and apex intensity of at least 10,000 (10x the input background noise level of 1000). Use the Find Peaks tab to inspect the blue (start) and red (end) boundary lines on a subset of 50 peaks and adjust smoothing/filter settings before committing. Each peak passing filtering is then manually reviewed in the Review Results tab, where boundary lines, the dashed RT line, and solid black lines marking incidental co-EIC peaks are inspected and labeled Good Peak, Bad Peak, or Multimodal/Indeterminate. Only 'good' adducts enter exported_metScribeR_library.csv, while the _with_metrics.csv retains all labels for auditing.

## Related tools

- **metScribeR** (R/Shiny package providing the Find Peaks, Review Results, and View/Export Library tabs for boundary visualization, manual peak labeling, and library export) — https://github.com/ncats/metScribeR
- **chromatographR** (supplies the find_peaks peak-detection routine applied to each adduct's EIC within metScribeR) — https://github.com/ethanbass/chromatographR
- **R** (runtime environment in which metScribeR is installed via devtools and the Shiny app is launched) — https://github.com/ncats/metScribeR
- **Shiny** (application framework hosting the interactive peak review and export workflow)

## Examples

```
devtools::install_github('ncats/metScribeR'); library(metScribeR); runMetScribeRShinyApp()
```

## Evaluation signals

- Every retained peak has at least 7 data points in a single EIC and a width at half maximum intensity strictly between 0 s and 15 s.
- No EIC yields more than 250 detected peaks, and no retained peak has apex intensity below 10,000 (10x the stated background noise of 1000).
- Boundary lines in review plots are plausible: blue line marks peak start, red line marks peak end, and the dashed line marks the RT; incidental co-EIC peaks appear as solid black lines.
- Exported_metScribeR_library.csv contains only 'good'-labeled adducts, while the _with_metrics.csv contains all three labels.
- metScribeR-derived RTs agree with manually assigned RTs (the article reports a correlation of 0.99 between manual and metScribeR-derived RTs).

## Limitations

- Manual review is still required: each peak passing filtering must be individually labeled, and m/z and RT tolerances must be chosen based on the LC-MS instrument's ability to confidently separate two signals.
- The 250-peaks-per-EIC cutoff was chosen as a generous but practical maximum for the authors' benchmarking data and may need adjustment for other datasets.
- For large submissions the initial computation is slow (~30 min for 12000 mzML files), and MoNA MS/MS lookups are dependent on server speed and can be slow.
- Only peaks reviewed manually are saved as PNGs in the Figures directory; peaks not reviewed leave no visual audit trail.
- Incorrectly spelled or cased column names in standards_df.csv or adduct_df.csv will cause an error.

## Evidence
