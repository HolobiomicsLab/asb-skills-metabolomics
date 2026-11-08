---
name: sample-metadata-table-construction
description: Use when preparing a GNPS FBMN/IIMN upload from UmetaFlow outputs and
  the quantification table needs a matching sample metadata file derived from `dataset.tsv`.
when_to_use_negative:
- When the user already supplies a finished GNPS metadata CSV exported from another
  pipeline and only needs to re-run networking, not rebuild the table.
- When the dataset lacks any sample annotations or blanks — the table would be a bare
  filename list, adding no grouping information.
- When the workflow is being used only for SIRIUS/MS2Query annotation without GNPS
  export, since the metadata table is only consumed by the GNPS upload.
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_3174
tools:
- name: UmetaFlow
  role: Generates the sample metadata (supplementary) table as part of the GNPS export
    sub-workflow, converting dataset.tsv into the comma-separated file uploaded with
    the MGF and quantification table
  repo: https://github.com/biosustain/snakemake_UmetaFlow
- name: Snakemake
  role: Orchestrates the rule that reads dataset.tsv/blanks.tsv and writes the GNPS
    metadata table in the UmetaFlow pipeline
  repo: https://github.com/biosustain/snakemake_UmetaFlow
- name: pyOpenMS
  role: Python bindings used by the UmetaFlow Jupyter variant to build the GNPS upload
    files, including the sample metadata table, from the processed feature tables
  repo: https://github.com/biosustain/pyOpenMS_UmetaFlow
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1186/s13321-023-00724-w
    title: 'UmetaFlow: an untargeted metabolomics workflow for high-throughput data
      processing and analysis'
  source_content_hash: sha256:3ecc610ff9a9c63e13a147786f47becedf2fd1a6edaa380d3753a2dbbb706641
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/biosustain/snakemake_UmetaFlow
  tools:
  - OpenMS
  - Python
  - Snakemake
  - UmetaFlow
  - pyOpenMS
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5625
n_claims: 16
---

# sample-metadata-table-construction

## Summary

Constructs the GNPS sample metadata table from UmetaFlow's `dataset.tsv` sample list, formatting filenames and sample attributes so that Feature-Based Molecular Networking and Ion Identity Molecular Networking jobs can group and compare samples correctly.

## When to use

Use when preparing a GNPS FBMN/IIMN upload from UmetaFlow outputs and the quantification table needs a matching sample metadata file derived from `dataset.tsv`.

## When NOT to use

- When the user already supplies a finished GNPS metadata CSV exported from another pipeline and only needs to re-run networking, not rebuild the table.
- When the dataset lacks any sample annotations or blanks — the table would be a bare filename list, adding no grouping information.
- When the workflow is being used only for SIRIUS/MS2Query annotation without GNPS export, since the metadata table is only consumed by the GNPS upload.

## Inputs

- config/dataset.tsv table listing sample_name and comment for each processed mzML/raw file
- config/blanks.tsv table of blank/QC sample names (optional)
- Set of mzML (or converted raw) files placed under data/mzML/ or data/raw/

## Outputs

- Comma-separated GNPS sample metadata table pairing each sample filename with its descriptive attributes
- One row per sample accepted for the GNPS FBMN/IIMN job, aligned with the Feature Quantification table columns

## How to apply

Within the UmetaFlow GNPS export sub-workflow, the metadata table is generated alongside the Feature Quantification table from the same sample list. The user first records each processed sample in `config/dataset.tsv` with its `sample_name` (exact mzML/raw filename, e.g. NBC_00162, ISP2_blank) and a free-text `comment` describing the biological condition (e.g. 'pyracrimicin', 'epemicins_A_B'). Blanks or QC files are kept out of the main sample table by listing them in `config/blanks.tsv`. The tooling then reads this tab-separated table and writes a comma-separated GNPS-compatible metadata file whose columns are sample filenames plus attributes, mirroring the supplementary table produced with the MGF, quantification table, and adduct-connection table. Correctness requires that every sample_name matches the feature quantification table's column headers exactly, since GNPS FBMN uses this mapping to associate feature intensities with sample attributes.

## Related tools

- **UmetaFlow** (Generates the sample metadata (supplementary) table as part of the GNPS export sub-workflow, converting dataset.tsv into the comma-separated file uploaded with the MGF and quantification table) — https://github.com/biosustain/snakemake_UmetaFlow
- **Snakemake** (Orchestrates the rule that reads dataset.tsv/blanks.tsv and writes the GNPS metadata table in the UmetaFlow pipeline) — https://github.com/biosustain/snakemake_UmetaFlow
- **pyOpenMS** (Python bindings used by the UmetaFlow Jupyter variant to build the GNPS upload files, including the sample metadata table, from the processed feature tables) — https://github.com/biosustain/pyOpenMS_UmetaFlow

## Examples

```
python data_files.py
```

## Evaluation signals

- Every sample_name in the generated metadata table appears verbatim as a column header in the GNPS Feature Quantification table.
- Blank/QC entries listed in blanks.tsv are excluded from the sample rows passed to GNPS.
- The metadata file is comma-separated, as required by GNPS FBMN/IIMN upload alongside the MGF and quantification table.
- Row count of the metadata table equals the number of sample files listed in dataset.tsv (after blank removal), matching the mzML file set.

## Limitations

- Metadata attributes come from the free-text 'comment' field of dataset.tsv, so inconsistent naming or misspelled condition labels will propagate to GNPS grouping.
- The workflow requires a manually maintained dataset.tsv; there is no automatic detection of sample conditions from raw filenames.
- If vendor files are converted outside the workflow (Agilent/Bruker via proteowizard), filenames must still match dataset.tsv exactly or the metadata mapping breaks.

## Evidence

- [readme] Convert the filtered consensus map into a GNPS Feature Quantification table (TXT) via pyOpenMS.: [evidence span withheld — anchor status: not_found]
- [readme] Add all your files in the data/raw/ or data/mzML/ directory and generate the dataset.tsv table to specify the samples (filenames) that will be processed.: "Add all your files in the `data/raw/` or `data/mzML/` directory and generate the `dataset.tsv` table to specify the samples (filenames) that will be processed."
- [readme] generate all the files necessary to create a FBMN or IIMN job at GNPS: [evidence span withheld — anchor status: external_doc]
