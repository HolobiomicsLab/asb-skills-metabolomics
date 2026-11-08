---
name: programmatic-pdf-report-generation
description: Use when tabulated MassQL query results (peak intensities per query per
  mzML file) need to be rendered into per-query visualization PDFs and summary csv
  tables for system suitability assessment of UHPLC-HRMS acquisitions.
when_to_use_negative:
- Input data has not been converted to mzML and convert_raw is not enabled (vendor
  .raw files must first go through ProteoWizard MSConvert)
- The analysis goal is feature finding or statistical metabolomics rather than visual
  reporting of MassQL query results (intensity and peak shape were not evaluated in
  this SST use case)
- The task requires assessing DDA MS/MS acquisition consistency, since the stochastic
  nature of MS/MS triggering limits SST conclusions for MS2 data
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: MassQLab
  role: Applies MassQL queries to a directory of mzML files, tabulates results, and
    saves them as images and csv files organized by timestamp
  repo: https://github.com/JohnsonDylan/MassQLab
- name: MassQL
  role: Domain-specific query language whose results (e.g., MS1MZ=207.1418:TOLERANCEPPM=2.5
    filters) supply the data being visualized in the report
  repo: https://github.com/mwang87/MassQueryLanguage
- name: matplotlib
  role: Renders EIC, Gaussian-fit, peak-area, and MS2 spectra plots for each named
    MassQL query per mzML file
- name: reportlab
  role: Compiles the per-query plots into a resultant PDF report (e.g., massqlab_report.pdf)
- name: openpyxl
  role: Formats and exports tabulated peak-area results as an .xlsx file
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1002/rcm.10132
    title: 'Open‐Source System Suitability: Mass Spectrometry Query Language Lab (MassQLab)'
  source_content_hash: sha256:ab34503f33d8c7b42aca0db5e600e479e66d2b5cd2aca67c89582ee0ba20f581
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/JohnsonDylan/MassQLab
  tools:
  - JupyterLab
  - MassQL
  - MassQLab
  - Python
  - matplotlib
  - openpyxl
  - reportlab
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# Programmatic PDF Report Generation

## Summary

Generates a structured report package from tabulated MassQL query results: per-query PDF plots of extracted ion chromatograms, Gaussian fits, peak areas, and MS2 product ion spectra, alongside csv/xlsx result tables, organized by timestamp in a MassQLab_Output directory. This automates system suitability testing (SST) reporting for LC-MS/LC-MS/MS data.

## When to use

Use when tabulated MassQL query results (peak intensities per query per mzML file) need to be rendered into per-query visualization PDFs and summary csv tables for system suitability assessment of UHPLC-HRMS acquisitions.

## When NOT to use

- Input data has not been converted to mzML and convert_raw is not enabled (vendor .raw files must first go through ProteoWizard MSConvert)
- The analysis goal is feature finding or statistical metabolomics rather than visual reporting of MassQL query results (intensity and peak shape were not evaluated in this SST use case)
- The task requires assessing DDA MS/MS acquisition consistency, since the stochastic nature of MS/MS triggering limits SST conclusions for MS2 data

## Inputs

- Directory of mzML mass spectrometry files (optionally converted from vendor .raw via MSConvert)
- MassQL query file (JSON, CSV, or XLSX) with 'name' and 'query' fields per query
- massqlab_config. specifying data_directory, queryfile, output_directory, and analysis flags

## Outputs

- ms1_traces.pdf and ms1_summary_traces.pdf (Gaussian-fitted EIC plots per query/file)
- ms1_summary_areas.pdf (peak areas per query per file)
- ms2_plots.pdf and ms2_summary_plots.pdf (MS2 scan intensity and product ion spectra plots)
- ms1_raw_df.csv, ms1_analysis_df.csv, ms1_RT_analysis_df.csv (MS1 raw and peak-fitting tables)
- ms2_raw_df.csv, ms2_analysis_df.csv (MS2 raw and peak-picking tables)
- ms1_consolidated_traces.png (consolidated image of all ms1 traces with Gaussian fits)
- Timestamped MassQLab_Output directory containing all artifacts

## How to apply

Load the MassQL query definitions (with user-defined 'name' and 'query' fields, from JSON, CSV, or XLSX per massqlab_config.) and apply each query to every mzML file in the data directory; raw results are tabulated to ms1_raw_df.csv and ms2_raw_df.csv. For MS1 queries, plot extracted ion chromatograms with Gaussian fits (output ms1_traces.pdf, ms1_summary_traces.pdf, ms1_summary_areas.pdf); for MS2 queries (e.g., precursor m/z 429.3765 with product ion m/z 85.0281 at 10 ppm tolerance), plot scan intensities and product ion spectra (ms2_plots.pdf, ms2_cluster_plots.pdf). Export analysis tables (ms1_analysis_df.csv, ms2_analysis_df.csv) for peak fitting/peak picking, and write all PDFs and csvs into MassQLab_Output under a timestamped folder inside the configured output_directory. Optionally compile per-query plots into a consolidated PDF report using matplotlib for plotting and reportlab/openpyxl for the PDF/xlsx assembly, as in the reconstructed per-run report generator workflow.

## Related tools

- **MassQLab** (Applies MassQL queries to a directory of mzML files, tabulates results, and saves them as images and csv files organized by timestamp) — https://github.com/JohnsonDylan/MassQLab
- **MassQL** (Domain-specific query language whose results (e.g., MS1MZ=207.1418:TOLERANCEPPM=2.5 filters) supply the data being visualized in the report) — https://github.com/mwang87/MassQueryLanguage
- **matplotlib** (Renders EIC, Gaussian-fit, peak-area, and MS2 spectra plots for each named MassQL query per mzML file)
- **reportlab** (Compiles the per-query plots into a resultant PDF report (e.g., massqlab_report.pdf))
- **openpyxl** (Formats and exports tabulated peak-area results as an .xlsx file)

## Examples

```
py src\MassQLab_console.py
```

## Evaluation signals

- massqlab_report.pdf (or ms1_traces.pdf / ms2_plots.pdf) exists and contains a plot section for each named MassQL query in the query file
- The exported csv/xlsx contains tabulated peak-area results for every query × mzML file combination (matching ms1_raw_df.csv / ms1_analysis_df.csv schema)
- All outputs reside in a timestamped MassQLab_Output directory inside the configured output_directory (data_directory by default)
- MS1 Gaussian-fit plots retain the query RT window (e.g., RTMIN=1.0 to RTMAX=1.2) and m/z tolerance metadata (e.g., 2.5 ppm)
- RT statistics table (ms1_RT_analysis_df.csv) reports RSD values consistent with prior runs (0.5%–2.3% across the SST analytes)

## Limitations

- Intensity and peak shape were not evaluated in the illustrative SST assessment, so the report does not support those metrics in that use case
- The stochastic nature of DDA MS/MS triggering is problematic for assessing consistency via SST, albeit normalization to total signal helped
- MS/MS SST evaluation may not be warranted for all use cases; MassQLab performance and available functions are still planned to be refined and expanded
- Raw-to-mzML conversion via convert_raw is experimental; converting with ProteoWizard MSConvert beforehand is recommended

## Evidence

- [intro] MassQLab applies a series of queries (written in the language of MassQL) to a directory containing mass spectrometry data in mzML format. Results are tabulated and saved as images and csv files.: "MassQLab applies a series of queries (written in the language of MassQL) to a directory containing mass spectrometry data in mzML format. Results are tabulated and saved as images and csv files."
- [readme] Results are saved in MassQLab_Output inside the defined output_directory (data_directory is used by default), organized by timestamp.: "Results are saved in `MassQLab_Output` inside the defined `output_directory` (`data_directory` is used by default), organized by timestamp."
- [methods] MassQLab is a Python-based Jupyter notebook implementation of MassQL.: [evidence span withheld — anchor status: not_found]
- [methods] MassQLab uses MassQL to apply a series of queries to a series of files and tabulates and visualizes the results.: "MassQLab uses MassQL to apply a series of queries to a series of files and tabulates and visualizes the results."
- [readme] Each query in the file must include a unique name and the MassQL query string itself.: "Each query in the file must include the following: - **`name`**: A unique name for each query (per MS1/MS2 type)."
