---
name: media-blank-background-component-removal
description: Use when a Progenesis-format peak table contains blank groups (solvent
  or media blanks) and features overrepresented in those blanks must be identified
  and removed to correct between-sample carryover and contamination.
when_to_use_negative:
- The dataset has no blank/control group — filter_group requires a specified group
  whose overrepresented features define background components.
- The goal is to correct mispicked isotopic peaks, non-reproducible ions across technical
  replicates, or in-source fragments — those require filter_mispicked_ions(), filter_cv(),
  or filter_insource_ions() instead.
- Input is raw MS/MS data rather than a pre-processed peak table; mpactr operates
  on pre-processed tandem MS/MS output such as Progenesis exports.
edam_operation: http://edamontology.org/operation_3803
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: mpactr
  role: Provides import_data(), filter_group(), filter_summary(), and get_peak_table()
    used to flag and remove features overrepresented in blank groups
  repo: https://github.com/mums2/mpactr
- name: R
  role: Execution environment for the mpactr package workflow
- name: data.table
  role: Format of the filtered peak table returned by get_peak_table() and written
    to CSV
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1128/mra.00997-24
    title: 'mpactR: an R adaptation of the metabolomics peak analysis computational
      tool (MPACT) for use in reproducible data analysis pipelines'
  source_content_hash: sha256:ac3ff4922e490bfe8bada052fa35854f8b845261e7f1836ad08be26c212b0ccf
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mums2/mpactr
  tools:
  - R
  - data.table
  - mpactr
  edam_operation: http://edamontology.org/operation_3803
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# media-blank-background-component-removal

## Summary

Removes background components from LC-MS/MS peak tables by flagging features whose abundance exceeds a user-defined threshold in a specified group of samples (e.g. media blanks or solvent blanks) using mpactr's filter_group().

## When to use

Use when a Progenesis-format peak table contains blank groups (solvent or media blanks) and features overrepresented in those blanks must be identified and removed to correct between-sample carryover and contamination.

## When NOT to use

- The dataset has no blank/control group — filter_group requires a specified group whose overrepresented features define background components.
- The goal is to correct mispicked isotopic peaks, non-reproducible ions across technical replicates, or in-source fragments — those require filter_mispicked_ions(), filter_cv(), or filter_insource_ions() instead.
- Input is raw MS/MS data rather than a pre-processed peak table; mpactr operates on pre-processed tandem MS/MS output such as Progenesis exports.

## Inputs

- Progenesis-format LC-MS/MS peak table (e.g. PTY087I2_dataset.csv)
- Sample metadata designating blank groups (e.g. solvent blanks, media blanks) alongside biological groups

## Outputs

- Filtered peak table (data.table) with background-component ions removed, exportable as e.g. blank_filtered_peak_table.csv
- Filter summary reporting counts of ions that failed and passed the group filter per group

## How to apply

Import the Progenesis-format peak table (e.g. PTY087I2_dataset.csv) and sample metadata with mpactr::import_data(format = "Progenesis"). Call mpactr::filter_group with group_to_remove set to the blank group (e.g. "Blanks" for solvent blanks or "Media" for media blanks) and a group_threshold (e.g. 0.01, meaning a feature is flagged if its relative abundance exceeds 1% in that group), with remove_ions = TRUE to actually drop flagged ions. Run filter_group once per blank group — in the PTY087I2 workflow it is applied twice, first for "Blanks" then for "Media" — since filters are independent and each call removes features overrepresented in one specific group. Verify performance with mpactr::filter_summary by checking counts of failed vs. passed ions, then extract the cleaned data.table with mpactr::get_peak_table(). The rationale is that solvent blank impurities cause carryover between samples, while background components are defined as features whose abundance is greater than the user-defined threshold in a specific sample group.

## Related tools

- **mpactr** (Provides import_data(), filter_group(), filter_summary(), and get_peak_table() used to flag and remove features overrepresented in blank groups) — https://github.com/mums2/mpactr
- **R** (Execution environment for the mpactr package workflow)
- **data.table** (Format of the filtered peak table returned by get_peak_table() and written to CSV)

## Examples

```
mp <- mpactr::filter_group(mpactr::import_data("PTY087I2_dataset.csv", format = "Progenesis"), group_threshold = 0.01, group_to_remove = "Media", remove_ions = TRUE)
```

## Evaluation signals

- filter_summary shows a nonzero count of ions flagged as failed for the blank/media group and the remaining passed-ion count matches the retained rows in the output peak table.
- No retained feature exceeds the group_threshold relative abundance (e.g. 1%) in the removed blank group.
- The extracted get_peak_table() output is a valid data.table whose rows correspond to surviving features and columns to samples in the metadata.
- Running filter_group for each blank group in sequence (e.g. "Blanks" then "Media") yields consistent ion counts that can be reproduced from the PTY087I2 dataset.

## Limitations

- The filter removes any feature whose abundance exceeds the threshold in the specified group, so an inappropriate group choice or overly permissive threshold may discard genuine biological features.
- All mpactr filters are independent; filter_group only addresses background components and does not correct mispicked peaks, replicate variability, or in-source fragmentation.
- Workflow assumes pre-processed Progenesis-format peak tables; raw instrument data must be pre-processed upstream.

## Evidence

- [methods] background components: features whose abundance is greater than user-defined abundance threshold in a specific group of samples: "background components: features whose abundance is greater than user-defined abundance threshold in a specific group of samples"
- [methods] solvent blank contamination removal: "solvent blank contamination: removal of features present in solvent blanks due to carryover between samples."
- [readme] filter_group removal of overrepresented features: [evidence span withheld — anchor status: external_doc]
- [readme] filters are independent: [evidence span withheld — anchor status: external_doc]
- [methods] PTY087I2 dataset description: "This dataset contain 38 samples for biological groups solvent blanks, media blanks, Streptomyces sp. PTY08712 grown with (250um_Ce) and without (0um_Ce) rare earth element cerium"
