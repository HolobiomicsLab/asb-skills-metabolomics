---
name: chromatographic-peak-quality-filtering
description: Use when processing LC-MS mzML files of authentic metabolite standards
  and you need to find peaks on adduct EICs and remove low-quality detections before
  assigning retention times and m/z for an in-house reference library.
when_to_use_negative:
- Input is already a curated library CSV or a feature table from targeted peak-picking
  tools such as MZmine, El-MAVEN, or MS-DIAL — peak finding and filtering have already
  been done.
- The goal requires MS/MS spectra for library construction — metScribeR deliberately
  focuses on MS1 and RT data and does not use or require MS2.
- Raw vendor files have not been converted to mzML; the noise parsing and EIC extraction
  pipeline expects mzML inputs.
tools:
- name: metScribeR
  role: R/Shiny package that creates EICs, wraps chromatographR peak finding, and
    applies the peak quality filtering thresholds
  repo: https://github.com/ncats/metScribeR
- name: chromatographR
  role: Provides the underlying find_peaks peak-detection routine applied to each
    adduct's EIC
  repo: ethanbass/chromatographR
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/acs.jproteome.5c00548
    title: 'MetScribeR: A Semiautomated Tool for Data Processing of In-House LC-MS
      Metabolite Reference Libraries'
  source_content_hash: sha256:43310d24583cdbe1d6a102c2a51bf8c02ea17e13e792e570221a974dd92fb043
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
  - chromatographR
  - metScribeR
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# chromatographic-peak-quality-filtering

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Filters peaks detected on extracted ion chromatograms (EICs) of metabolite standard adducts using chromatographR-based peak finding plus metScribeR quality thresholds, so only credible peaks survive for RT and m/z library construction.

## When to use

Use when processing LC-MS mzML files of authentic metabolite standards and you need to find peaks on adduct EICs and remove low-quality detections before assigning retention times and m/z for an in-house reference library.

## When NOT to use

- Input is already a curated library CSV or a feature table from targeted peak-picking tools such as MZmine, El-MAVEN, or MS-DIAL — peak finding and filtering have already been done.
- The goal requires MS/MS spectra for library construction — metScribeR deliberately focuses on MS1 and RT data and does not use or require MS2.
- Raw vendor files have not been converted to mzML; the noise parsing and EIC extraction pipeline expects mzML inputs.

## Inputs

- Positive and negative ESI LC-MS raw data converted to mzML format (e.g. via MSConvert)
- standards_df.csv with columns common_name, monoisotopic_mass, pos_mode_mzML_file_path, neg_mode_mzML_file_path (optionally inchiKey, additional_identifiers)
- adduct_df.csv with columns adduct, change_from_neutral, mode (POS or NEG)
- User-set background noise level, m/z tolerance, and RT tolerance
- Peak tables from chromatographR-based find_eic_peaks on each adduct EIC

## Outputs

- Filtered peak table of surviving peaks per EIC (filtered_peak_table.csv in the reconstructed workflow)
- Per-peak boundary assignments (start and end boundaries, dashed RT line) rendered for visual inspection
- Descriptive boxplots of peak metrics (points per peak, width at half maximum, intensity) to guide threshold selection

## How to apply

First build EICs for each standard adduct with metScribeR's create_eic/import_eic_data, using m/z and RT tolerances chosen to match the ability of the LC-MS instrument to separate two signals. Detect peaks on each EIC with the chromatographR-derived find_eic_peaks routine, toggling the density filter and data smoothing to find the best settings for the data. Then apply the quality filters: retain peaks with at least 7 data points in a single EIC, a width at half maximum intensity greater than 0 s and less than 15 s, no more than 250 peaks per EIC (a generous practical cutoff for benchmarking data), and an apex intensity of at least 10,000, i.e. 10 times the background noise level of 1000 read from the noise plot. Exclude the background noise level itself at the mzML parsing stage, below which all MS observations are dropped from processing. Finally, inspect representative filtered peaks with the peak plotting functions to confirm that the blue (start) and red (end) boundary lines are correctly assigned before downstream adduct conflict checking and manual Good Peak/Bad Peak review.

## Related tools

- **metScribeR** (R/Shiny package that creates EICs, wraps chromatographR peak finding, and applies the peak quality filtering thresholds) — https://github.com/ncats/metScribeR
- **chromatographR** (Provides the underlying find_peaks peak-detection routine applied to each adduct's EIC) — ethanbass/chromatographR

## Examples

```
devtools::install_github('ethanbass/chromatographR'); BiocManager::install("mzR"); devtools::install_github('ncats/metScribeR'); metScribeR::runMetScribeRShinyApp()
```

## Evaluation signals

- Every retained peak has at least 7 data points within a single EIC.
- Width at half maximum intensity of each retained peak is > 0 s and < 15 s.
- No EIC contributes more than 250 peaks to the filtered table.
- No retained peak has an apex intensity below 10,000 (10x the 1000-level background noise).
- Spot-check representative peaks with the peak_demo_plot/peak_plot functions and verify the blue/red boundary lines and dashed RT line fall on plausible chromatographic features.

## Limitations

- The 250 peaks-per-EIC cutoff is a 'generous but practical maximum' tuned for the authors' benchmarking data and may need adjustment for other datasets.
- The 15 s width-at-half-maximum bound reflects the authors' specific LC-MS method; practitioners must set filtering criteria based on their own LC-MS method knowledge and the Descriptive Boxplots tab.
- MS/MS retrieval from MoNA is dependent on server speed and can often be slow (relevant only to the downstream library-export step).
- Performance scales with submission size: initial computation took about 30 minutes for 12,000 mzML files.

## Evidence
