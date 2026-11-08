---
name: lc-ms-eic-peak-detection
description: Use when processing mzML files of authentic metabolite standards to detect
  and quality-filter peaks per adduct EIC before assigning retention times and building
  an RT/m/z reference library.
when_to_use_negative:
- Input is already a feature/peak table from targeted peak-picking tools such as MZmine,
  El-MAVEN, or MS-DIAL — this skill operates on raw mzML-derived EICs
- Library construction requires MS/MS-based identification rather than MS1 + RT data;
  metScribeR deliberately does not use MS2 for library building
- The data are experimental (unknown) samples rather than in-house authentic metabolite
  standard runs
edam_operation: http://edamontology.org/operation_3215
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: metScribeR
  role: R package whose create_eic/import_eic_data and find_eic_peaks routines perform
    EIC extraction, peak finding, and threshold-based filtering on adduct chromatograms
  repo: https://github.com/ncats/metScribeR
- name: chromatographR
  role: Upstream dependency supplying the find_peaks peak-detection engine used on
    each adduct's EIC
  repo: https://github.com/ethanbass/chromatographR
- name: mzR
  role: Bioconductor package installed to parse the mzML files during EIC construction
- name: R
  role: Runtime environment for metScribeR and the Shiny application driving the peak-detection
    workflow
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/acs.jproteome.5c00548
    title: 'MetScribeR: A Semiautomated Tool for Data Processing of In-House LC-MS
      Metabolite Reference Libraries'
  source_content_hash: sha256:628c2f02e735cc320173fafcc7542ac17421d9ad63bb981c514b1b41d3da83b7
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
  - mzR
  edam_operation: http://edamontology.org/operation_3215
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# lc-ms-eic-peak-detection

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Peak finding and quality filtering on extracted ion chromatograms (EICs) for LC-MS metabolite standard data, as implemented in metScribeR using chromatographR's find_peaks with user-defined smoothing, density-filter, and threshold settings.

## When to use

Use when processing mzML files of authentic metabolite standards to detect and quality-filter peaks per adduct EIC before assigning retention times and building an RT/m/z reference library.

## When NOT to use

- Input is already a feature/peak table from targeted peak-picking tools such as MZmine, El-MAVEN, or MS-DIAL — this skill operates on raw mzML-derived EICs
- Library construction requires MS/MS-based identification rather than MS1 + RT data; metScribeR deliberately does not use MS2 for library building
- The data are experimental (unknown) samples rather than in-house authentic metabolite standard runs

## Inputs

- mzML files of authentic standards run in positive and negative ESI modes
- standards_df.csv with columns common_name, monoisotopic_mass, pos_mode_mzML_file_path, neg_mode_mzML_file_path (optionally inchiKey, additional_identifiers)
- adduct_df.csv with columns adduct, change_from_neutral, mode (POS/NEG)
- user-defined background noise level, m/z and RT tolerances, and peak-filtering thresholds

## Outputs

- filtered_peak_table.csv of peaks that survived the 7-point, 0-15 s width, 250-peaks-per-EIC, and 10000-intensity thresholds
- Per-EIC peak boundaries (start/end boundary lines) for manual review and peak_demo_plot/peak_plot inspection
- Updated metScribeR storage_object.RDS carrying peak-detection state

## How to apply

First build EICs for each adduct from the mzML files using metScribeR's create_eic/import_eic_data with the m/z and RT tolerances chosen for the LC-MS method, and remove MS observations below the user-entered background noise level (e.g. 1000). Run the chromatographR-derived find_eic_peaks routine on each EIC with user-tuned smoothing and density-filter settings. Filter candidate peaks against four thresholds: at least 7 data points per peak in a single EIC, width at half maximum intensity greater than 0 s and less than 15 s, no more than 250 peaks per EIC (a generous practical maximum for the benchmarking data), and intensity of at least 10,000 (10x the background noise level of 1000). Write the surviving peaks to filtered_peak_table.csv and inspect representative peaks with peak_demo_plot/peak_plot to confirm the blue start boundary and red end boundary were assigned sensibly before manual review.

## Related tools

- **metScribeR** (R package whose create_eic/import_eic_data and find_eic_peaks routines perform EIC extraction, peak finding, and threshold-based filtering on adduct chromatograms) — https://github.com/ncats/metScribeR
- **chromatographR** (Upstream dependency supplying the find_peaks peak-detection engine used on each adduct's EIC) — https://github.com/ethanbass/chromatographR
- **mzR** (Bioconductor package installed to parse the mzML files during EIC construction)
- **R** (Runtime environment for metScribeR and the Shiny application driving the peak-detection workflow)

## Examples

```
library(metScribeR); runMetScribeRShinyApp()  # then use the Find Peaks tab to tune density filter/smoothing and submit thresholds
```

## Evaluation signals

- Retained peaks have >= 7 data points per EIC, FWHM width > 0 s and < 15 s, and intensity >= 10,000 (10x the 1000 background noise level)
- No EIC contributes more than 250 candidate peaks (the stated generous practical maximum for the benchmarking data)
- Peak boundaries are visually plausible in peak_demo_plot/peak_plot output: blue line marks the peak start, red line the peak end, dashed line marks the RT
- Manual review concordance: metScribeR-derived RTs showed a correlation of 0.99 with manual RTs and poor-quality peaks were appropriately filtered out

## Limitations

- m/z and RT tolerances and all four filtering thresholds must be set based on the analyst's knowledge of the specific LC-MS method; defaults may not transfer across instruments or gradients
- The 250-peaks-per-EIC cutoff was chosen as a generous but practical maximum only for the benchmarking data
- Peaks must still pass single-click manual review (Good Peak / Bad Peak / Multimodal/Indeterminate) before library inclusion; filtering is not fully automatic
- Large submissions are computationally slow (~30 min for 12000 mzML files) at the initial submit step

## Evidence
