---
name: relative-abundance-threshold-background-removal
description: Use when a Progenesis-format LC-MS/MS peak table includes solvent blank
  or media blank sample groups and features overrepresented in those blanks (e.g.
  carryover) must be identified and removed before downstream analysis.
when_to_use_negative:
- The samples lack blank/background control groups (e.g. no solvent blanks or media
  blanks), so there is no group against which to threshold abundance.
- 'The goal is a different QC issue mpactr handles with its own filters: mispicked
  isotopic patterns (filter_mispicked_ions), non-reproducible features across technical
  replicates (filter_cv), or in-source fragment ions (filter_insource_ions).'
- Input is raw tandem MS/MS data rather than a pre-processed peak table importable
  via import_data().
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: mpactr
  role: R package providing import_data(), filter_group(), filter_summary(), and get_peak_table()
    used to threshold and remove background ions from blank groups
  repo: https://github.com/mums2/mpactr
- name: R
  role: Runtime environment for the mpactr package and its filter workflow
  repo: https://github.com/mums2/mpactr/actions/workflows/r.yml
- name: mpact
  role: Original Python MPACT tool that mpactR's group filter adapts; source of the
    PTY087I2 raw data used in the vignette
  repo: https://github.com/BalunasLab/mpact
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1128/mra.00997-24
    title: 'mpactR: an R adaptation of the metabolomics peak analysis computational
      tool (MPACT) for use in reproducible data analysis pipelines'
  source_content_hash: sha256:1c7f8ede54559bf1e109681b7133472d97aa362d17a76beb210317fe4c18ba84
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mums2/mpactr
  tools:
  - R
  - mpact
  - mpactr
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6153846153846154
n_claims: 13
---

# relative-abundance-threshold-background-removal

## Summary

Applies mpactr's filter_group() to identify background components — features whose abundance exceeds a user-defined relative-abundance threshold in a specified sample group — and remove them, e.g. solvent blank impurities causing carryover and contamination in LC-MS/MS peak tables.

## When to use

Use when a Progenesis-format LC-MS/MS peak table includes solvent blank or media blank sample groups and features overrepresented in those blanks (e.g. carryover) must be identified and removed before downstream analysis.

## When NOT to use

- The samples lack blank/background control groups (e.g. no solvent blanks or media blanks), so there is no group against which to threshold abundance.
- The goal is a different QC issue mpactr handles with its own filters: mispicked isotopic patterns (filter_mispicked_ions), non-reproducible features across technical replicates (filter_cv), or in-source fragment ions (filter_insource_ions).
- Input is raw tandem MS/MS data rather than a pre-processed peak table importable via import_data().

## Inputs

- Progenesis-format LC-MS/MS peak table (e.g. PTY087I2_dataset.csv)
- Sample metadata with a column defining sample groups (e.g. solvent blanks, media blanks, biological groups)
- mpactr peakTable object produced by import_data()

## Outputs

- mpactr peakTable object with background ions flagged and, with remove_ions = TRUE, removed
- Filtered peak table (data.table) extractable via get_peak_table(), e.g. written to blank_filtered_peak_table.csv
- Filter summary report of failed and passed ion counts per group (filter_summary)

## How to apply

Import the Progenesis-format peak table (e.g. PTY087I2_dataset.csv) and metadata with mpactr::import_data(format = "Progenesis"). Call filter_group() with a group_to_remove target (e.g. "Blanks" or "Media") and a relative-abundance group_threshold such as 0.01, flagging ions whose abundance exceeds the threshold in that group; set remove_ions = TRUE to drop them. Because filters are independent, run filter_group once per background group (first solvent blanks, then media blanks) to build a project-specific workflow. Verify performance with filter_summary by checking counts of failed and passed ions per group, then extract the filtered data.table with get_peak_table(). The rationale is that features present in solvent blanks arise from between-sample carryover and contamination, and their removal yields high-quality MS1 features.

## Related tools

- **mpactr** (R package providing import_data(), filter_group(), filter_summary(), and get_peak_table() used to threshold and remove background ions from blank groups) — https://github.com/mums2/mpactr
- **R** (Runtime environment for the mpactr package and its filter workflow) — https://github.com/mums2/mpactr/actions/workflows/r.yml
- **mpact** (Original Python MPACT tool that mpactR's group filter adapts; source of the PTY087I2 raw data used in the vignette) — https://github.com/BalunasLab/mpact

## Examples

```
library(mpactr); data <- import_data("PTY087I2_dataset.csv", metadata, format = "Progenesis"); blank_filtered <- filter_group(data, group_to_remove = "Blanks", remove_ions = TRUE); write.csv(get_peak_table(blank_filtered), "blank_filtered_peak_table.csv")
```

## Evaluation signals

- filter_summary shows a nonzero count of ions failed by the group filter for the target group (e.g. Blanks) and the expected count of passed ions after removal.
- No retained feature in the output peak table exceeds the group_threshold (e.g. 0.01) relative abundance in the removed group (Blanks/Media).
- get_peak_table() returns a data.table whose columns match the imported Progenesis features minus the removed background ions; row count decreased accordingly.
- Applying filter_group twice (Blanks then Media) each reports its own failed/passed ion counts, confirming independent filter application per background group.
- import_data(format = "Progenesis") succeeds and get_metadata() returns the sample groups used as group_to_remove targets, confirming metadata-to-peak-table alignment.

## Limitations

- The filter only targets features overrepresented in a user-specified group above the chosen threshold; features just below the threshold or background present across groups rather than concentrated in one group will not be flagged.
- The threshold (e.g. group_threshold = 0.01) is user-defined and dataset-dependent; an inappropriate value can over- or under-remove features.
- Filters in mpactr are independent, so background removal does not correct mispicked peaks, replicate CV issues, or in-source fragments; those require the other filters.

## Evidence

- [methods] background components: features whose abundance is greater than user-defined abundance threshold in a specific group of samples: "background components: features whose abundance is greater than user-defined abundance threshold in a specific group of samples"
- [methods] solvent blank impurities: "Removing solvent blank impurities is important for correcting for between-sample carryover and contamination in experimental samples. You can identify and remove these ions with mpactr's"
- [methods] solvent blank contamination: "solvent blank contamination: removal of features present in solvent blanks due to carryover between samples."
- [readme] filter_group README: "`filter_group()`: removal of features overrepresented in a specific
group of samples; for example removal of features present in solvent
blanks due to carryover between samples."
- [readme] filters are independent: "All filters are independent, meaning they can be used to create a
project-specific workflow"
- [methods] get_peak_table: "If you want to extract the filtered peak table, with filters that have been applied, use `get_peak_table()`"
- [methods] import_data: "To import these data into R, use the mpactr function `import_data()`"
