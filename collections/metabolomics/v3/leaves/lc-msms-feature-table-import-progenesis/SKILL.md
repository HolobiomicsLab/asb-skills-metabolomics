---
name: lc-msms-feature-table-import-progenesis
description: Use when you have tandem MS/MS pre-processed feature tables exported
  in Progenesis format (a peak table CSV plus a sample metadata CSV) and need to load
  them into R as an mpactr object for quality filtering.
when_to_use_negative:
- Data are not in Progenesis pre-processed peak-table format (e.g. raw vendor files
  or mzML spectra) — mpactr expects pre-processed feature tables, not raw tandem MS/MS
  data
- You only need to inspect or summarize an already-filtered mpactr object (e.g. call
  `qc_summary()` or `get_peak_table()`) without re-importing
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_0121
tools:
- name: mpactr
  role: Provides `import_data()` to load Progenesis peak table and metadata into R
    and construct the mpactr object used by all filters
  repo: https://github.com/mums2/mpactr
- name: R
  role: Runtime environment in which the mpactr package (installed from CRAN or GitHub)
    is executed
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1128/mra.00997-24
    title: 'mpactR: an R adaptation of the metabolomics peak analysis computational
      tool (MPACT) for use in reproducible data analysis pipelines'
  source_content_hash: sha256:a9aef965b817521a01331d788e2b129634abf28650682ac1190b7d91cfa4289e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mums2/mpactr
  tools:
  - R
  - mpactr
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# lc-msms-feature-table-import-progenesis

## Summary

Imports Progenesis-formatted LC-MS/MS peak tables and metadata into R via the mpactr `import_data()` function so that downstream mpactr filters (mispicked ions, group filters, CV filtering, in-source ion removal) can operate on the feature table.

## When to use

Use when you have tandem MS/MS pre-processed feature tables exported in Progenesis format (a peak table CSV plus a sample metadata CSV) and need to load them into R as an mpactr object for quality filtering.

## When NOT to use

- Data are not in Progenesis pre-processed peak-table format (e.g. raw vendor files or mzML spectra) — mpactr expects pre-processed feature tables, not raw tandem MS/MS data
- You only need to inspect or summarize an already-filtered mpactr object (e.g. call `qc_summary()` or `get_peak_table()`) without re-importing

## Inputs

- Progenesis-format LC-MS/MS peak table CSV (e.g. cultures_peak_table.csv)
- Sample metadata CSV matching the peak table (e.g. cultures_metadata.csv)

## Outputs

- mpactr object holding the imported peak table and metadata, ready for filter functions

## How to apply

Load the bundled example data (e.g. `cultures_peak_table.csv` and `cultures_metadata.csv`) or your own Progenesis-exported peak table and metadata using the mpactr function `import_data()`, which creates the mpactr object that all subsequent filters operate on. The peak table must contain MS1 feature intensities across all samples, and the metadata must carry sample group information (e.g. Solvent_Blank, Media, and biological groups such as 0um_Ce and 250um_Ce) so that group-based filters like `filter_group()` can identify features overrepresented in blank groups. Confirm the import succeeds by checking that the resulting object exposes the peak table via `get_peak_table()` and metadata via `get_metadata()`. Only after a successful import should the filter chain (`filter_mispicked_ions(merge_peaks=TRUE, merge_method='sum')`, `filter_group()`, `filter_cv(cv_threshold=0.2)`, optionally `filter_insource_ions()`) be applied, since `qc_summary()` and other reporting functions rely on the imported object.

## Related tools

- **mpactr** (Provides `import_data()` to load Progenesis peak table and metadata into R and construct the mpactr object used by all filters) — https://github.com/mums2/mpactr
- **R** (Runtime environment in which the mpactr package (installed from CRAN or GitHub) is executed)

## Examples

```
mpactr_obj <- mpactr::import_data(peak_table = 'cultures_peak_table.csv', metadata = 'cultures_metadata.csv')
```

## Evaluation signals

- The mpactr object returned by `import_data()` exposes a peak table via `get_peak_table()` whose columns match the input Progenesis CSV features and rows match samples
- `get_metadata()` returns sample group labels consistent with the metadata CSV, enabling group-based filters such as `filter_group()` on Solvent_Blank and Media groups
- Applying the filter chain (`filter_mispicked_ions`, `filter_group`, `filter_cv(cv_threshold=0.2)`) and then `qc_summary()` yields a data.table with one row per input compound id and a non-empty status column
- The import does not error and downstream `get_group_averages()` / `get_cv_data()` calls return values consistent with the imported feature intensities

## Limitations

- mpactr operates on pre-processed LC-MS/MS peak tables; it does not itself handle raw acquisition files or instrument-specific conversion
- The paper's demonstrated workflow uses a bundled dataset (PTY087I2, 38 samples across solvent blanks, media blanks, and biological groups with and without cerium) — import assumptions (column layout, group naming) follow that Progenesis export
- The mpactr README lists filters as independent, but the import step itself has no tunable parameters described, so malformed or non-Progenesis CSVs will fail silently or produce unusable objects

## Evidence

- [methods] import_data: "To import these data into R, use the mpactr function `import_data()`"
- [other] Progenesis peak table and metadata inputs: [evidence span withheld — anchor status: not_found]
- [readme] filter prerequisites and group naming: [evidence span withheld — anchor status: not_found]
- [methods] dataset used for demonstration: "This dataset contain 38 samples for biological groups solvent blanks, media blanks, Streptomyces sp. PTY08712 grown with (250um_Ce) and without (0um_Ce) rare earth element cerium"
- [abstract] mpactr scope on pre-processed data: "mpactR automates pre-processing of liquid chromatography-tandem mass spectrometry (LC-MS/MS) data from microbiological samples"
