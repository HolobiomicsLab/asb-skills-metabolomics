---
name: lc-msms-feature-table-filtering
description: Use when you have a Progenesis-format LC-MS/MS peak table plus sample
  metadata and need to correct mispicked isotope peaks, remove solvent-blank/media
  background, CV-filter technical replicates, or drop in-source fragments.
when_to_use_negative:
- Your peak table is not from tandem MS/MS pre-processing (e.g. it is already a curated,
  quality-filtered feature table with no replicate, blank, or isotope issues to correct)
- You need raw tandem MS/MS file conversion or feature detection/decomconvolution
  itself — mpactr starts from an already pre-processed Progenesis peak table
- You only need visualization or correlation of sample profiles (ggplot/plotly feature-failure
  plots, Hmisc/corrplot correlations, ggdendro clustering) without applying filters
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: mpactR
  role: Provides import_data() and the independent filters filter_mispicked_ions(),
    filter_group(), filter_cv(), and filter_insource_ions() plus accessors get_peak_table(),
    get_cv_data(), filter_summary(), get_similar_ions(), and get_metadata() applied
    to the imported peak table
  repo: https://github.com/mums2/mpactr
- name: R
  role: Runtime environment for the mpactr package, installed from CRAN or GitHub
    with devtools::install_github('mums2/mpactr')
  repo: https://github.com/mums2/mpactr
- name: MPACT
  role: Original Python tool whose filters mpactR adapts for reproducible R pipelines
  repo: https://github.com/BalunasLab/mpact
- name: data.table
  role: Format of the filtered peak tables and similar-ions records extracted via
    get_peak_table() and get_similar_ions()
provenance:
  source_task_ids:
  - task_002
  - task_004
  - task_005
  source_papers:
  - doi: 10.1128/mra.00997-24
    title: 'mpactR: an R adaptation of the metabolomics peak analysis computational
      tool (MPACT) for use in reproducible data analysis pipelines'
  source_content_hash: sha256:e317a4999f5a91699d7b5f575d9ff53b93cfc968fd7565351798f42844869145
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mums2/mpactr
  tools:
  - MPACT
  - R
  - data.table
  - mpactR
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5217391304347826
n_claims: 23
---

# lc-msms-feature-table-filtering

## Summary

Filter an imported LC-MS/MS peak table with the R package mpactr to correct mispicked isotope peaks, remove blank/background components, non-reproducible technical-replicate ions, and in-source fragments, yielding a high-quality MS1 feature table.

## When to use

Use when you have a Progenesis-format LC-MS/MS peak table plus sample metadata and need to correct mispicked isotope peaks, remove solvent-blank/media background, CV-filter technical replicates, or drop in-source fragments.

## When NOT to use

- Your peak table is not from tandem MS/MS pre-processing (e.g. it is already a curated, quality-filtered feature table with no replicate, blank, or isotope issues to correct)
- You need raw tandem MS/MS file conversion or feature detection/decomconvolution itself — mpactr starts from an already pre-processed Progenesis peak table
- You only need visualization or correlation of sample profiles (ggplot/plotly feature-failure plots, Hmisc/corrplot correlations, ggdendro clustering) without applying filters

## Inputs

- Progenesis-format LC-MS/MS peak table (e.g. PTY087I2_dataset.csv)
- Sample metadata CSV (e.g. PTY087I2_samplelist.csv joined with extract metadata, filtered to injections present in the peak table)
- mpactr object created by import_data()

## Outputs

- Filtered peak table (data.table) from get_peak_table(), e.g. blank_filtered_peak_table.csv or cv_filtered_peak_table.csv
- filter_summary lists of failed_ions and passed_ions per filter
- Similar-ions record from get_similar_ions() listing each retained main ion and its merged similar ions
- Per-ion coefficient of variation from get_cv_data() (e.g. cv_values.csv)
- Group means from get_group_averages() on the filtered compounds

## How to apply

Import the Progenesis peak table and metadata into R with mpactr::import_data(peak_table, metadata, format = 'Progenesis') to create the mpactr object. Correct mispicked isotopic patterns with filter_mispicked_ions(ringwin = 0.5, isowin = 0.01, trwin = 0.005, max_iso_shift = 3, merge_peaks = TRUE, merge_method = 'sum'), which groups similar ions and merges them by summing intensities. Remove background components with filter_group(group_threshold = 0.01, group_to_remove = 'Blanks' and then 'Media', remove_ions = TRUE), dropping ions whose relative abundance exceeds 1% in the blank groups. Filter non-reproducible ions with filter_cv(cv_threshold = 0.2, copy_object = TRUE) and optionally insource fragments with filter_insource_ions(). Filters are independent, so order them in a project-specific workflow, then extract the reduced feature table with get_peak_table() and record outcomes with filter_summary(obj, 'mispicked'), get_similar_ions(), and get_cv_data(). Use copy_object = FALSE when you want reference semantics updating the object in place rather than copying.

## Related tools

- **mpactR** (Provides import_data() and the independent filters filter_mispicked_ions(), filter_group(), filter_cv(), and filter_insource_ions() plus accessors get_peak_table(), get_cv_data(), filter_summary(), get_similar_ions(), and get_metadata() applied to the imported peak table) — https://github.com/mums2/mpactr
- **R** (Runtime environment for the mpactr package, installed from CRAN or GitHub with devtools::install_github('mums2/mpactr')) — https://github.com/mums2/mpactr
- **MPACT** (Original Python tool whose filters mpactR adapts for reproducible R pipelines) — https://github.com/BalunasLab/mpact
- **data.table** (Format of the filtered peak tables and similar-ions records extracted via get_peak_table() and get_similar_ions())

## Examples

```
library(mpactr); data <- import_data(peak_table, metadata, format = 'Progenesis'); data <- filter_mispicked_ions(data, ringwin = 0.5, isowin = 0.01, trwin = 0.005, max_iso_shift = 3, merge_peaks = TRUE, merge_method = 'sum', copy_object = TRUE); ft <- get_peak_table(data)
```

## Evaluation signals

- Ion count invariant: number of ions in get_peak_table() after filter_mispicked_ions equals pre-filter ion count minus the failed_ions count reported by filter_summary(obj, 'mispicked')
- Filtered feature table and the get_similar_ions() record are non-empty data.table objects
- filter_summary returns plausible failed_ions/passed_ions counts for both the Blanks and Media group removals at group_threshold = 0.01
- get_cv_data() returns one CV value per ion consistent with cv_threshold = 0.2 filtering (retained ions should have CV at or below the threshold across technical replicates)
- All four filters are independent — applying them in a different order or subset should not error, and each get_peak_table() export reflects exactly the filters applied

## Limitations

- Filters assume the input peak table is already pre-processed (Progenesis format); mpactr corrects peak-selection errors but does not perform raw-file feature detection
- The mispicked-ion merge merges similar ions by summing intensities (merge_method = 'sum'), so intensities of merged features are combined rather than kept separate
- With copy_object = FALSE the filters operate on reference semantics, updating the mpactr object in place — a copy must be requested explicitly via copy_object = TRUE to preserve the unfiltered object
- Group filtering with group_threshold = 0.01 removes any ion exceeding 1% relative abundance in the specified group, which may discard genuine features overrepresented in a non-blank group if the wrong group is named
- The original MPACT Python tool (Rev 23.05.15) notes its GNPS peak table filtering is experimental and only tested with FBMN export in MS-DIAL

## Evidence

- [methods] Mispicked ions are corrected with specific wins/shift parameters: "mpactr provides filters to correct for the following issues observed during preprocessing of tandem MS/MS data:"
- [methods] mispicked ions definition: "mispicked ions: isotopic patterns that are incorrectly split during preprocessing."
- [methods] background components definition: "background components: features whose abundance is greater than user-defined abundance threshold in a specific group of samples"
- [methods] CV filter for technical replicates: "Non-reproducible ions are identified by coefficient of variation (CV) between technical replicates with `filter_cv()`"
- [readme] README overview of the four independent filters: [evidence span withheld — anchor status: external_doc]
- [readme] README states filters are independent: [evidence span withheld — anchor status: external_doc]
