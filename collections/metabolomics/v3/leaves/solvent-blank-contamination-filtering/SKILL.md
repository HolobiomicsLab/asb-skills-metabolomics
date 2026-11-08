---
name: solvent-blank-contamination-filtering
description: Use when a Progenesis-format LC-MS/MS peak table includes blank control
  groups (e.g., solvent blanks, media blanks) and experimental samples may carry contamination
  or carryover that must be removed before downstream analysis.
when_to_use_negative:
- The input is raw MS/MS spectra rather than a pre-processed Progenesis peak table
  with sample-group metadata.
- The dataset has no blank or control group to compare against, so no group is overrepresented
  by carryover.
- The quality issue is mispicked isotopic peaks or non-reproducible features across
  technical replicates, which mpactr addresses with filter_mispicked_ions() and filter_cv()
  instead.
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0092
tools:
- name: mpactr
  role: Provides filter_group(), filter_summary(), get_peak_table(), and import_data()
    used to threshold-and-remove blank-group ions from the peak table
  repo: https://github.com/mums2/mpactr
- name: mpact
  role: Original Python MPACT implementation whose blank/background-removal logic
    mpactr's filter_group() adapts for R pipelines
  repo: https://github.com/BalunasLab/mpact
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1128/mra.00997-24
    title: 'mpactR: an R adaptation of the metabolomics peak analysis computational
      tool (MPACT) for use in reproducible data analysis pipelines'
  source_content_hash: sha256:fe9cb2bc67930bb8de43d45dbdcabd1115b50d8173c6b421d161bc7928b59fa8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mums2/mpactr
  tools:
  - R
  - mpact
  - mpactr
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 12
---

# solvent-blank-contamination-filtering

## Summary

Identifies and removes background components—such as solvent-blank carryover and media-blank contaminants—from LC-MS/MS peak tables using mpactr's filter_group(), which flags features whose abundance exceeds a user-defined threshold in a specified sample group.

## When to use

Use when a Progenesis-format LC-MS/MS peak table includes blank control groups (e.g., solvent blanks, media blanks) and experimental samples may carry contamination or carryover that must be removed before downstream analysis.

## When NOT to use

- The input is raw MS/MS spectra rather than a pre-processed Progenesis peak table with sample-group metadata.
- The dataset has no blank or control group to compare against, so no group is overrepresented by carryover.
- The quality issue is mispicked isotopic peaks or non-reproducible features across technical replicates, which mpactr addresses with filter_mispicked_ions() and filter_cv() instead.

## Inputs

- Progenesis-format LC-MS/MS peak table (CSV, e.g., PTY087I2_dataset.csv) with ion abundances per sample
- Sample metadata mapping samples to biological groups, including blank groups (e.g., solvent blanks, media blanks)

## Outputs

- Filtered peak table (data.table) with background ions removed, exported e.g. to blank_filtered_peak_table.csv
- Filter summary of failed/passed ion counts for each filtered group (via filter_summary)

## How to apply

Load the Progenesis-format peak table (e.g., PTY087I2_dataset.csv) and sample metadata with mpactr::import_data(format = "Progenesis"). Call mpactr::filter_group with group_to_remove set to the blank group (e.g., "Blanks") and group_threshold = 0.01, meaning any ion whose relative abundance exceeds 1% in that group is flagged as a background component; set remove_ions = TRUE to drop them. Repeat the call for other background groups present in the experiment, such as "Media" media blanks, since the PTY087I2 dataset contains both solvent blanks and media blanks alongside Streptomyces sp. PTY087I2 grown with and without cerium. Check performance with mpactr::filter_summary, which reports counts of failed and passed ions per filter, then extract the cleaned data.table via mpactr::get_peak_table(). The rationale is that features overrepresented in blanks indicate carryover between injections or media contamination rather than true biological signal.

## Related tools

- **mpactr** (Provides filter_group(), filter_summary(), get_peak_table(), and import_data() used to threshold-and-remove blank-group ions from the peak table) — https://github.com/mums2/mpactr
- **mpact** (Original Python MPACT implementation whose blank/background-removal logic mpactr's filter_group() adapts for R pipelines) — https://github.com/BalunasLab/mpact

## Examples

```
mpactr::filter_group(data, group_threshold = 0.01, group_to_remove = "Blanks", remove_ions = TRUE)
```

## Evaluation signals

- filter_summary reports non-zero failed-ion counts for the Blanks and Media filters and the remaining ions pass the 0.01 group threshold.
- No retained ion in the filtered peak table exceeds 1% relative abundance in the designated blank group.
- get_peak_table() returns a data.table whose row count equals the input ion count minus the ions flagged as background components.
- The filtered peak table exports cleanly to CSV (e.g., blank_filtered_peak_table.csv) and retains sample columns matching the imported metadata.

## Limitations

- The group threshold (e.g., 0.01) is user-defined, so overly permissive values can retain blank contaminants while overly strict values can discard genuine low-abundance features.
- Each background group must be filtered in a separate filter_group() call; the filter targets one group_to_remove at a time.
- The filter only addresses overrepresentation within a named sample group; it does not correct mispicked peaks, in-source fragments, or replicate variability.

## Evidence

- [methods] background components: "background components: features whose abundance is greater than user-defined abundance threshold in a specific group of samples"
- [methods] solvent blank contamination: "Removing solvent blank impurities is important for correcting for between-sample carryover and contamination in experimental samples."
- [readme] filter_group README: "removal of features overrepresented in a specific
  group of samples; for example removal of features present in solvent
  blanks due to carryover between samples."
