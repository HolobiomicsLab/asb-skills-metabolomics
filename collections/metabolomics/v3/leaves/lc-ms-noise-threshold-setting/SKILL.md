---
name: lc-ms-noise-threshold-setting
description: Use when processing LC-MS mzML files of authentic metabolite standards
  in metScribeR and a background noise level must be chosen from the noise plot before
  peak finding and filtering.
when_to_use_negative:
- Input data are already processed peak/feature tables rather than raw mzML files
  where noise level must be estimated
- The analysis goal is MS/MS-based library construction; metScribeR and its noise
  threshold workflow explicitly do not use or require MS2 data
- Background noise must be quantified automatically across many studies without user
  judgment; this step is a manual, user-input-driven decision in metScribeR
edam_operation: http://edamontology.org/operation_3218
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: metScribeR
  role: R package / Shiny app whose initial submission screen displays the noise plot
    from which the background noise level is read and entered before EIC creation
  repo: https://github.com/ncats/metScribeR
- name: chromatographR
  role: Dependency providing EIC/peak-detection routines that consume the noise-filtered
    signal downstream
  repo: https://github.com/ethanbass/chromatographR
- name: MSConvert
  role: Optional converter of vendor MS data into .mzML, the format metScribeR's noise-threshold
    workflow expects
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/acs.jproteome.5c00548
    title: 'MetScribeR: A Semiautomated Tool for Data Processing of In-House LC-MS
      Metabolite Reference Libraries'
  source_content_hash: sha256:1b67aeb46bd4e7453d7ad80589511c2921b879c6c1d22da2b3844dc93d980d7b
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
  - MSConvert
  - R
  - chromatographR
  - metScribeR
  edam_operation: http://edamontology.org/operation_3218
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# lc-ms-noise-threshold-setting

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Guides the setting of the background noise threshold in the metScribeR Shiny app so that MS observations below a user-defined intensity level are excluded before peak finding on extracted ion chromatograms.

## When to use

Use when processing LC-MS mzML files of authentic metabolite standards in metScribeR and a background noise level must be chosen from the noise plot before peak finding and filtering.

## When NOT to use

- Input data are already processed peak/feature tables rather than raw mzML files where noise level must be estimated
- The analysis goal is MS/MS-based library construction; metScribeR and its noise threshold workflow explicitly do not use or require MS2 data
- Background noise must be quantified automatically across many studies without user judgment; this step is a manual, user-input-driven decision in metScribeR

## Inputs

- standards_df.csv (or .tsv) with columns common_name, monoisotopic_mass, pos_mode_mzML_file_path, neg_mode_mzML_file_path
- adduct_df.csv (or .tsv) with columns adduct, change_from_neutral, mode (POS/NEG)
- mzML files for positive and negative ESI mode acquisitions

## Outputs

- User-defined background noise level entered into the metScribeR app
- Filtered peak table (filtered_peak_table.csv) in which peaks below the noise-derived intensity cutoff have been removed

## How to apply

Launch the metScribeR Shiny app, upload standards_df.csv (columns common_name, monoisotopic_mass, pos_mode_mzML_file_path, neg_mode_mzML_file_path) and adduct_df.csv (columns adduct, change_from_neutral, mode), and inspect the noise plot shown on the initial submission screen to read off the level of background noise in the data. Input that level so that all MS observations below it are excluded from processing; in the paper's benchmarking data this level was 1000, and peaks with intensity less than 10,000 (10 times the background noise level) were subsequently excluded as a peak-filtering step. Then set m/z and RT tolerances for creating EICs based on the LC-MS instrument's ability to confidently separate two signals, and click submit to trigger the initial computation (~30 min for 12000 mzML files).

## Related tools

- **metScribeR** (R package / Shiny app whose initial submission screen displays the noise plot from which the background noise level is read and entered before EIC creation) — https://github.com/ncats/metScribeR
- **chromatographR** (Dependency providing EIC/peak-detection routines that consume the noise-filtered signal downstream) — https://github.com/ethanbass/chromatographR
- **MSConvert** (Optional converter of vendor MS data into .mzML, the format metScribeR's noise-threshold workflow expects)

## Examples

```
devtools::install_github('ncats/metScribeR'); runMetScribeRShinyApp()
```

## Evaluation signals

- The chosen noise level matches the visual baseline evident in the noise plot generated on the initial submission screen
- Peak-filtering threshold is consistent with the paper's ratio rule: minimum apex intensity of 10,000 corresponds to 10× the background noise level of 1000
- No surviving peaks in filtered_peak_table.csv have intensity at or below the stated noise level
- Downstream find_peaks (chromatographR-derived) runs successfully on EICs built with the user-supplied m/z and RT tolerances

## Limitations

- The noise level is a manual estimate from the noise plot; metScribeR does not state an automated noise-estimation algorithm, so misjudging the baseline shifts the 10× intensity cutoff
- m/z and RT tolerances must also be set in the same step and depend on the instrument's ability to confidently separate two signals
- Initial computation is slow (~30 min for 12000 mzML files), so a poorly chosen noise level forces repeating the expensive submission

## Evidence
