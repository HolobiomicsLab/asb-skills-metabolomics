---
name: chromatogram-smoothing-parameter-tuning
description: Use when building an in-house LC-MS RT/m/z library with metScribeR and
  EIC peaks (from mzML files of authentic standards) show poorly assigned start/end
  boundaries or noisy baselines, requiring tuning of smoothing and density-filter
  settings before applying peak-finding thresholds.
when_to_use_negative:
- Input is already a filtered peak table or feature table rather than raw mzML chromatograms
  needing smoothing adjustment
- The workflow requires MS/MS-based identification — metScribeR library construction
  uses only MS1 and RT data and does not require or use MS2 spectra
- A fully unattended peak-finding pipeline is needed; this step is deliberately interactive
  and requires visual human judgment on a peak subset
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0092
tools:
- name: metScribeR
  role: R/Shiny package whose 'Find Peaks' tab implements the interactive density-filter
    and smoothing toggles used to tune peak detection parameters on EICs
  repo: https://github.com/ncats/metScribeR
- name: chromatographR
  role: R package installed from ethanbass/chromatographR that metScribeR builds on
    for chromatogram smoothing and peak finding on EICs
  repo: https://github.com/ethanbass/chromatographR
- name: R
  role: Runtime language for metScribeR; the Shiny app is launched via the runMetScribeRShinyApp()
    function
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/acs.jproteome.5c00548
    title: 'MetScribeR: A Semiautomated Tool for Data Processing of In-House LC-MS
      Metabolite Reference Libraries'
  source_content_hash: sha256:84d0b24b31901a893080d1eabe065811ac6a6b6eaad8e337f0cae3e6d0b3cadf
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
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# chromatogram-smoothing-parameter-tuning

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

An interactive parameter-tuning step in metScribeR's Shiny app where the user visually adjusts the density filter and data smoothing on EIC peaks before committing filtering thresholds, ensuring peak boundaries are correctly assigned for LC-MS metabolite standard library construction.

## When to use

Use when building an in-house LC-MS RT/m/z library with metScribeR and EIC peaks (from mzML files of authentic standards) show poorly assigned start/end boundaries or noisy baselines, requiring tuning of smoothing and density-filter settings before applying peak-finding thresholds.

## When NOT to use

- Input is already a filtered peak table or feature table rather than raw mzML chromatograms needing smoothing adjustment
- The workflow requires MS/MS-based identification — metScribeR library construction uses only MS1 and RT data and does not require or use MS2 spectra
- A fully unattended peak-finding pipeline is needed; this step is deliberately interactive and requires visual human judgment on a peak subset

## Inputs

- standards_df.csv (or .tsv) with columns common_name, monoisotopic_mass, pos_mode_mzML_file_path, neg_mode_mzML_file_path
- adduct_df.csv (or .tsv) with columns adduct, change_from_neutral, mode (POS/NEG)
- mzML files from positive and negative ESI LC-MS runs of authentic standards
- Background noise level, m/z tolerance, and RT tolerance entered in the Shiny app
- metScribeR storage_object.RDS from a prior session (optional)

## Outputs

- Tuned density-filter and data-smoothing settings for peak finding
- Filtered peak table with peak boundaries (start/end), apex RT, and intensity per adduct EIC
- Descriptive boxplots summarizing peak properties across the dataset
- Subset of 50 example EIC peak plots rendered in the Find Peaks tab

## How to apply

After submitting standards_df.csv, adduct_df.csv, and mzML files with the background noise level (below which all MS observations are excluded) plus m/z and RT tolerances, move to the 'Find Peaks' tab of the Shiny app. Cycle through a drop-down menu of a subset of 50 detected peaks and toggle the density filter and data smoothing settings, visually inspecting where the blue (start) and red (end) vertical boundary lines are drawn on each EIC. Base the final filtering criteria on three sources: your knowledge of the LC-MS method, the tuned peak figures, and the boxplots on the Descriptive Boxplots tab. Then set the concrete thresholds — at least 7 data points per peak in a single EIC, width at half maximum intensity between 0 s and 15 s (0–0.25 min), a maximum of 250 peaks per EIC, and minimum apex intensity of 10,000 (10x the 1000 background noise level) — and click the submit button to apply peak finding across all adduct EICs.

## Related tools

- **metScribeR** (R/Shiny package whose 'Find Peaks' tab implements the interactive density-filter and smoothing toggles used to tune peak detection parameters on EICs) — https://github.com/ncats/metScribeR
- **chromatographR** (R package installed from ethanbass/chromatographR that metScribeR builds on for chromatogram smoothing and peak finding on EICs) — https://github.com/ethanbass/chromatographR
- **R** (Runtime language for metScribeR; the Shiny app is launched via the runMetScribeRShinyApp() function)

## Examples

```
devtools::install_github('ncats/metScribeR'); library(metScribeR); setwd('metScribeR_extdata_folder'); runMetScribeRShinyApp()
```

## Evaluation signals

- Peak start/end boundaries (blue/red vertical lines) fall on sensible points of each EIC peak rather than on noise or baseline drift
- Surviving peaks satisfy the configured thresholds: ≥7 data points per peak in a single EIC, 0 s < width at half maximum < 15 s, apex intensity ≥ 10,000
- No more than 250 peaks per EIC are retained (the generous practical maximum for the benchmarking data)
- Representative peaks re-inspected with peak_demo_plot/peak_plot confirm correct boundary assignment after filtering
- Poor-quality peaks are appropriately filtered out, and surviving peaks show a correlation of ~0.99 with manually assigned RTs in the benchmark

## Limitations

- Parameter tuning is semiautomated and subjective — settings depend on user inspection of the 50-peak subset and the Descriptive Boxplots tab, not a fixed rule
- Thresholds must be set per LC-MS method; the defaults (7 points, 0–15 s FWHM, 250 peaks/EIC, 10,000 intensity) were tuned for the authors' benchmarking data
- The initial computation is slow at scale (~30 min for 12,000 mzML files), so tuning mistakes require reloading storage_object.RDS to avoid repeating the full computation

## Evidence
