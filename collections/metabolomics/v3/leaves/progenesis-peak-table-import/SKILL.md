---
name: progenesis-peak-table-import
description: Use when raw LC-MS/MS pre-processing output is a Progenesis-format peak
  table CSV that must be joined with sample metadata (Injection, Sample_Code, Biological_Group)
  before applying mpactR filters.
when_to_use_negative:
- Input is raw vendor/centroided spectra rather than a post-preprocessing MS1 feature
  table — mpactR corrects peak-selection errors introduced during preprocessing, so
  it expects an exported peak table, not spectra files.
- The data are already loaded in an mpactR object — apply filters (filter_mispicked_ions,
  filter_cv, filter_group, filter_insource_ions) directly instead of re-importing.
- Peak lists exported from MS-DIAL or Bruker Metaboscape, which the original Python
  MPACT supports, are not covered by the documented Progenesis import of mpactR.
edam_operation: http://edamontology.org/operation_0306
tools:
- name: mpactr
  role: Provides import_data() to parse the Progenesis peak table and metadata CSVs
    into an R6 object, plus get_metadata(), get_raw_data(), and get_peak_table() to
    access the stored tables
  repo: https://github.com/mums2/mpactr
- name: R
  role: Runtime for the import step; mpactr is installed with install.packages("mpactr")
    from CRAN or devtools::install_github("mums2/mpactr")
- name: data.table
  role: get_metadata(), get_raw_data(), and get_peak_table() return data.table objects
    used for import validation
provenance:
  source_task_ids:
  - task_001
  - task_002
  - task_005
  source_papers:
  - doi: 10.1128/mra.00997-24
    title: 'mpactR: an R adaptation of the metabolomics peak analysis computational
      tool (MPACT) for use in reproducible data analysis pipelines'
  source_content_hash: sha256:bee5ae361de9bad8e351bb02dcb74d0920aee037299aeeb0f13c68bfaee79dbf
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mums2/mpactr
  tools:
  - R
  - data.table
  - mpactr
  edam_operation: http://edamontology.org/operation_0306
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# progenesis-peak-table-import

## Summary

mpactR's import_data() ingests a Progenesis-format LC-MS/MS peak table CSV plus a sample metadata CSV into an mpactR R6 object, storing both tables for downstream filtering. It is the entry step of the mpactR reproducible pre-processing pipeline for high-quality MS1 features.

## When to use

Use when raw LC-MS/MS pre-processing output is a Progenesis-format peak table CSV that must be joined with sample metadata (Injection, Sample_Code, Biological_Group) before applying mpactR filters.

## When NOT to use

- Input is raw vendor/centroided spectra rather than a post-preprocessing MS1 feature table — mpactR corrects peak-selection errors introduced during preprocessing, so it expects an exported peak table, not spectra files.
- The data are already loaded in an mpactR object — apply filters (filter_mispicked_ions, filter_cv, filter_group, filter_insource_ions) directly instead of re-importing.
- Peak lists exported from MS-DIAL or Bruker Metaboscape, which the original Python MPACT supports, are not covered by the documented Progenesis import of mpactR.

## Inputs

- Progenesis-format peak table CSV (e.g. cultures_peak_table.csv or PTY087I2_dataset.csv) with compound, m/z, retention time, and raw abundance columns
- Sample metadata CSV containing required columns Injection, Sample_Code, Biological_Group (e.g. cultures_metadata.csv, or PTY087I2_samplelist.csv joined with PTY087I2_extractmetadata.csv and filtered to injections present in the peak table)
- R session with mpactr installed from CRAN or GitHub (mums2/mpactr)

## Outputs

- mpactR R6 class object storing the peak table and metadata
- get_metadata() → metadata as a data.table (Injection, Sample_Code, Biological_Group)
- get_raw_data() → unfiltered Progenesis peak table as a data.table
- get_peak_table() → peak table (data.table) ready for the mpactR filter steps
- Serialised import object such as imported_data.rds

## How to apply

Call mpactr::import_data() with peak_table_file_path and metadata_file_path set to the CSV paths and format = "Progenesis"; this returns an mpactR R6 class object storing both the peak table and the metadata. For a dry run, resolve the bundled examples via example_path() (cultures_peak_table.csv, cultures_metadata.csv); for a real dataset such as PTY087I2, first join PTY087I2_samplelist.csv with PTY087I2_extractmetadata.csv and restrict the merged metadata to injections present in the peak table before import. Validate the object with get_metadata() — confirming the required Injection, Sample_Code, Biological_Group columns — and get_raw_data() — confirming the unfiltered Progenesis table with compound, m/z, retention time, and raw abundance columns; both return data.table objects. Retrieve the peak table with get_peak_table() and persist the object with saveRDS (e.g. imported_data.rds) so downstream filters (filter_mispicked_ions, filter_cv, filter_group, filter_insource_ions) operate on a stable import.

## Related tools

- **mpactr** (Provides import_data() to parse the Progenesis peak table and metadata CSVs into an R6 object, plus get_metadata(), get_raw_data(), and get_peak_table() to access the stored tables) — https://github.com/mums2/mpactr
- **R** (Runtime for the import step; mpactr is installed with install.packages("mpactr") from CRAN or devtools::install_github("mums2/mpactr"))
- **data.table** (get_metadata(), get_raw_data(), and get_peak_table() return data.table objects used for import validation)

## Examples

```
mpactr::import_data(peak_table_file_path = "cultures_peak_table.csv", metadata_file_path = "cultures_metadata.csv", format = "Progenesis")
```

## Evaluation signals

- get_metadata() on the imported object returns a data.table containing the required Injection, Sample_Code, and Biological_Group columns.
- get_raw_data() returns the unfiltered Progenesis peak table with compound, m/z, retention time, and raw abundance columns matching the input CSV.
- get_peak_table() returns a non-empty data.table retrievable from the R6 object (e.g. get_peak_table(data2) in the reconstruction task).
- The returned object is the mpactR R6 class and round-trips through saveRDS/readRDS (imported_data.rds).
- Sample columns of the peak table align with injections listed in the metadata, which was filtered to injections present in the peak table.

## Limitations

- Only format = "Progenesis" is documented for import_data(); MS-DIAL and Bruker Metaboscape import support exists in the original Python MPACT, not in mpactR's importer.
- Metadata must contain the required Injection, Sample_Code, Biological_Group columns and be restricted to injections present in the peak table (as done for PTY087I2 by joining the samplelist with the extract metadata).
- The import produces an R6 reference object; downstream filters with copy_object = FALSE mutate it in place, so the pre-filter import state is only preserved via a copy or a saved .rds.

## Evidence

- [methods] import_data is the mpactR function to bring data into R: "To import these data into R, use the mpactr function `import_data()`"
- [methods] metadata access from the imported object: "Metadata can be accessed with `get_metadata()`"
- [methods] peak table extraction from the imported object: "If you want to extract the filtered peak table, with filters that have been applied, use `get_peak_table()`"
- [methods] PTY087I2 example dataset imported with Progenesis format: "This dataset contain 38 samples for biological groups solvent blanks, media blanks, Streptomyces sp. PTY08712 grown with (250um_Ce) and without (0um_Ce) rare earth element cerium"
- [abstract] mpactr targets pre-processed LC-MS/MS data: "mpactR automates pre-processing of liquid chromatography-tandem mass spectrometry (LC-MS/MS) data from microbiological samples"
- [readme] mpactr scope: MS1 features from tandem MS preprocessing: [evidence span withheld — anchor status: external_doc]
