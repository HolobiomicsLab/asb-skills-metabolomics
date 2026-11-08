---
name: lc-ms-peak-quality-filtering
description: Use when processing raw mzML files of LC-MS metabolite standards to decide
  which adduct peaks (per EIC) are high-quality enough for RT/m/z library inclusion,
  and when benchmarking automated filtering against manual Good/Bad Peak labels.
when_to_use_negative:
- Input is already a picked feature/peak table (e.g., MZmine, El-MAVEN, or MS-DIAL
  output) rather than raw mzML files needing EIC extraction and peak finding
- Library construction requires MS/MS spectra as a prerequisite — metScribeR focuses
  on MS1 and RT data and does not require MS2 for filtering or library building
- Data are large-scale untargeted study samples (e.g., ~12000 mzML files) rather than
  a manageable set of authentic standard runs
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_0091
tools:
- name: metScribeR
  role: R package with Shiny app that implements EIC extraction, peak finding, density
    filtering/smoothing, peak-quality filtering, and manual Good/Bad Peak review
  repo: https://github.com/ncats/metScribeR
- name: Shiny
  role: Web UI hosting the Find Peaks tab where filter criteria (data points, width
    at half max, peaks per EIC, intensity cutoff) are set
- name: R
  role: Language for reproducing the filter decisions, confusion counts, and precision/recall/F1
    metrics from the library-with-metrics CSV
- name: chromatographR
  role: Dependency installed alongside metScribeR for chromatogram processing
  repo: ethanbass/chromatographR
provenance:
  source_task_ids:
  - task_003
  - task_006
  source_papers:
  - doi: 10.1021/acs.jproteome.5c00548
    title: 'MetScribeR: A Semiautomated Tool for Data Processing of In-House LC-MS
      Metabolite Reference Libraries'
  source_content_hash: sha256:cf5cf2323bf45197db4be001e1bbaaacc4077077ace4ade63d7446f2d142906c
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
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9285714285714286
n_claims: 14
---

# lc-ms-peak-quality-filtering

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Automated peak-quality filtering of extracted ion chromatograms (EICs) from LC-MS runs of authentic metabolite standards, as implemented in metScribeR, to separate good peaks from bad ones before manual review. Benchmark on 860 IROA standard peaks shows precision 0.93, recall 0.98, and F1 0.95 against manual quality labels.

## When to use

Use when processing raw mzML files of LC-MS metabolite standards to decide which adduct peaks (per EIC) are high-quality enough for RT/m/z library inclusion, and when benchmarking automated filtering against manual Good/Bad Peak labels.

## When NOT to use

- Input is already a picked feature/peak table (e.g., MZmine, El-MAVEN, or MS-DIAL output) rather than raw mzML files needing EIC extraction and peak finding
- Library construction requires MS/MS spectra as a prerequisite — metScribeR focuses on MS1 and RT data and does not require MS2 for filtering or library building
- Data are large-scale untargeted study samples (e.g., ~12000 mzML files) rather than a manageable set of authentic standard runs

## Inputs

- mzML files of LC-MS runs of authentic metabolite standards (positive and negative ESI modes)
- standards_df.csv with columns common_name, monoisotopic_mass, pos_mode_mzML_file_path, neg_mode_mzML_file_path (optional inchiKey)
- adduct_df.csv with columns adduct, change_from_neutral, mode
- Background noise level read from the noise plot (e.g., 1000)
- exported_metScribeR_library_with_metrics.csv with manual 'Good Peak'/'Bad Peak' labels (for benchmarking)

## Outputs

- Filtered set of candidate peaks per adduct EIC flagged as good/bad by the automated criteria
- exported_metScribeR_library_with_metrics.csv retaining all good, bad, and multimodal/indeterminate adducts with peak metrics and manual labels
- filtering_performance_metrics.csv with precision, recall, F1, and FP/FN/TN confusion counts (benchmark task)
- Descriptive boxplots of peak metrics to guide criterion selection

## How to apply

Load mzML files (positive and negative ESI) into the metScribeR Shiny app with a standards_df.csv (common_name, monoisotopic_mass, pos_mode_mzML_file_path, neg_mode_mzML_file_path) and an adduct_df.csv (adduct, change_from_neutral, mode), then submit with m/z and RT tolerances set from instrument separation capability. On the 'Find Peaks' tab, tune the density filter and data smoothing, then set filtering criteria: minimum 7 data points per peak in a single EIC, width at half maximum intensity between 0 s and 15 s, a maximum of 250 peaks per EIC, and minimum peak intensity of 10,000 (10 times the background noise level of 1000, entered from the noise plot). In the benchmark, automated 'good' calls are treated as predictions against manual 'Good Peak'/'Bad Peak' clicks as ground truth, excluding 'Multimodal/Indeterminate' peaks from the positive/negative tally. Compute confusion counts (TP, FP, FN, TN) and derive precision, recall, and combined F1; the rationale is that narrow, adequately sampled, above-noise peaks in a single EIC reliably reproduce human quality judgments (0.93/0.98/0.95). Peaks passing filtering still require single-click manual review before library export, and additional adducts must elute within a user-defined RT tolerance of the assigned RT.

## Related tools

- **metScribeR** (R package with Shiny app that implements EIC extraction, peak finding, density filtering/smoothing, peak-quality filtering, and manual Good/Bad Peak review) — https://github.com/ncats/metScribeR
- **Shiny** (Web UI hosting the Find Peaks tab where filter criteria (data points, width at half max, peaks per EIC, intensity cutoff) are set)
- **R** (Language for reproducing the filter decisions, confusion counts, and precision/recall/F1 metrics from the library-with-metrics CSV)
- **chromatographR** (Dependency installed alongside metScribeR for chromatogram processing) — ethanbass/chromatographR

## Examples

```
devtools::install_github('ncats/metScribeR'); library(metScribeR); runMetScribeRShinyApp()
```

## Evaluation signals

- Against manual labels for 860 peaks (743 adducts), recomputed precision (0.93), recall (0.98), F1 (0.95), and counts FP=43, FN=10, TN=226 must exactly match the benchmark in filtering_performance_metrics.csv
- Every retained peak satisfies all four criteria: ≥7 data points per peak, width at half maximum intensity in (0 s, 15 s), ≤250 peaks per EIC, intensity ≥10,000
- exported_metScribeR_library_with_metrics.csv contains good, bad, and multimodal/indeterminate adducts with peak metrics and manual quality labels
- Peaks passing automated filtering should correlate with manual 'Good Peak' clicks; only ~43 of 860 peaks should be false positives under the benchmark criteria
- Additional adducts of a standard elute within the user-defined RT tolerance of the assigned RT

## Limitations

- Filtering criteria (7 data points, 0–15 s width, 250 peaks per EIC, 10× noise intensity) were chosen for the benchmarking LC-MS data and should be adjusted to the user's method; metScribeR exposes them as configurable parameters
- 43 of 860 peaks were false positives (passed filtering but manually low quality), so automated filtering cannot replace single-click manual review of every passing peak
- Peaks labeled 'Multimodal/Indeterminate' must be excluded from the positive/negative tally when benchmarking, or metrics will not match
- Crossed-adduct conflicts and RT tolerance assignments add constraints beyond raw filtering

## Evidence
