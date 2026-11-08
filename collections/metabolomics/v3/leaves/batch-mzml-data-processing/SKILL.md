---
name: batch-mzml-data-processing
description: Use when multiple mzML files need the same set of structured MS1/MS2
  queries (e.g., extracted-ion chromatograms, precursor/product-ion, or neutral-loss
  filters) applied in batch with tabulated peak areas and plots as outputs.
when_to_use_negative:
- Input data is in vendor-specific format and cannot first be converted to mzML —
  MassQL requires the open mzML format before use
- A single ad hoc query on one file is needed — the direct MassQL Python API (msql_engine.process_query)
  or command line tool may be simpler
- Intensity and peak-shape evaluation is required as a primary metric — the illustrative
  SST assessment explicitly did not evaluate intensity or peak shape
edam_operation: http://edamontology.org/operation_3216
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: MassQLab
  role: Python 3.9 Jupyter/console pipeline that applies each named MassQL query in
    the query file to every mzML file in the data directory and tabulates/visualizes
    results
  repo: https://github.com/JohnsonDylan/MassQLab
- name: MassQL
  role: Domain-specific query language used to express MS1 and MS2 filters (MS1MZ,
    MS2PREC, MS2PROD, MS2NL with ppm tolerances and RT windows) executed by MassQLab
  repo: https://github.com/mwang87/MassQueryLanguage
- name: MSConvert
  role: ProteoWizard tool used to convert vendor .raw files to mzML format with default
    settings before MassQL querying
  repo: https://proteowizard.sourceforge.io/
provenance:
  source_task_ids:
  - task_001
  - task_004
  source_papers:
  - doi: 10.1002/rcm.10132
    title: 'Open‐Source System Suitability: Mass Spectrometry Query Language Lab (MassQLab)'
  source_content_hash: sha256:5395a791ba56c4d653264367ca6e5d8ffef90a109b1aad5ab397cc5950ff7bce
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/JohnsonDylan/MassQLab
  tools:
  - MSConvert
  - MassQL
  - MassQLab
  - Python
  edam_operation: http://edamontology.org/operation_3216
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# batch-mzml-data-processing

## Summary

Apply a batch of MassQL queries to an entire directory of mzML mass spectrometry files using MassQLab, which tabulates and visualizes results (csv files and PDF/PNG images) for system suitability testing or targeted signal interrogation.

## When to use

Use when multiple mzML files need the same set of structured MS1/MS2 queries (e.g., extracted-ion chromatograms, precursor/product-ion, or neutral-loss filters) applied in batch with tabulated peak areas and plots as outputs.

## When NOT to use

- Input data is in vendor-specific format and cannot first be converted to mzML — MassQL requires the open mzML format before use
- A single ad hoc query on one file is needed — the direct MassQL Python API (msql_engine.process_query) or command line tool may be simpler
- Intensity and peak-shape evaluation is required as a primary metric — the illustrative SST assessment explicitly did not evaluate intensity or peak shape

## Inputs

- Directory of mzML mass spectrometry files (data_directory in massqlab_config.)
- Query file (MassQL_Queries., .csv, or .xlsx) with name + query fields per MassQL query
- Optionally vendor .raw files, converted to mzML with MSConvert default settings
- massqlab_config. configuration file (data_directory, queryfile, output_directory, analysis, cache_setting, convert_raw, msconvertexe)

## Outputs

- ms1_raw_df.csv and ms1_analysis_df.csv (raw MS1 query results and peak fitting analysis)
- ms1_RT_analysis_df.csv (retention-time statistics vs. query RT window)
- ms1_traces.pdf, ms1_summary_traces.pdf, ms1_summary_areas.pdf and PNG/PDF summary plots
- ms2_raw_df.csv and ms2_analysis_df.csv (raw MS2 query results and peak picking analysis)
- ms2_plots.pdf, ms2_summary_plots.pdf, ms2_cluster_plots.pdf outputs
- All results saved in MassQLab_Output inside the output_directory (data_directory by default), organized by timestamp

## How to apply

Convert any vendor .raw files to open mzML format using MSConvert with default settings (or set convert_raw=true with an msconvertexe path, noting this is experimental). Populate massqlab_config. with data_directory pointing to the mzML files and queryfile pointing to a JSON/CSV/XLSX query file where each query has a unique 'name' and the MassQL 'query' string (e.g., QUERY scaninfo(MS1DATA) FILTER MS1MZ=207.1418:TOLERANCEPPM=2.5 AND RTMIN=1.0 AND RTMAX=1.2, or MS2 queries with MS2PREC/MS2PROD/MS2NL filters at specified ppm tolerances). Run the full pipeline via src/MassQLab_console.py or notebooks/MassQLab_console.ipynb, which applies every named query to every file in the data directory; MassQLab was developed in Python 3.9 and built on the MassQL engine. With analysis=true, downstream peak fitting yields extracted chromatographic peak areas, exported as csv files (ms1_raw_df.csv, ms1_analysis_df.csv, ms2_raw_df.csv, etc.) plus PDF summary plots, saved in MassQLab_Output organized by timestamp. Enable cache_setting=true to store an intermediate Feather file for faster re-querying.

## Related tools

- **MassQLab** (Python 3.9 Jupyter/console pipeline that applies each named MassQL query in the query file to every mzML file in the data directory and tabulates/visualizes results) — https://github.com/JohnsonDylan/MassQLab
- **MassQL** (Domain-specific query language used to express MS1 and MS2 filters (MS1MZ, MS2PREC, MS2PROD, MS2NL with ppm tolerances and RT windows) executed by MassQLab) — https://github.com/mwang87/MassQueryLanguage
- **MSConvert** (ProteoWizard tool used to convert vendor .raw files to mzML format with default settings before MassQL querying) — https://proteowizard.sourceforge.io/

## Examples

```
py src\MassQLab_console.py
```

## Evaluation signals

- Each detected peak must meet the detection criteria to be considered valid in MassQLab downstream analysis
- Raw and analysis csv outputs exist and are non-empty for both MS1 (ms1_raw_df.csv, ms1_analysis_df.csv) and MS2 (ms2_raw_df.csv, ms2_analysis_df.csv)
- Every query in the query file has a unique name (per MS1/MS2 type); duplicated or overly long names cause output issues
- Retention-time RSD of SST analytes should fall in the reproducible range reported for this workflow (0.5% to 2.3%)
- Results directory MassQLab_Output is created inside the defined output_directory and organized by timestamp, containing the expected csv and PDF files

## Limitations

- Vendor-specific file formats must be converted to mzML before MassQL use; MassQLab's built-in convert_raw feature is experimental and MSConvert is recommended
- MassQLab processes all files in the data_directory — there is no per-file selection within a directory
- MassQL currently tests in Python 3.9; other versions may not work
- Query names must be unique per MS1/MS2 type and should avoid very long names to prevent output issues
- MassQLab performance and available functions are still to be refined/expanded, and no software version number is provided

## Evidence

- [intro] MassQLab applies a series of queries (written in the language of MassQL) to a directory containing mass spectrometry data in mzML format. Results are tabulated and saved as images and csv files.: "MassQLab applies a series of queries (written in the language of MassQL) to a directory containing mass spectrometry data in mzML format. Results are tabulated and saved as images and csv files."
- [methods] Raw data files were converted to mzML format using MSConvert with default settings.: "Raw data files (.raw) were converted to mzML format using MSConvert (https://proteowizard.sourceforge.io/) with default settings."
- [methods] MassQLab is a Python-based Jupyter notebook implementation of MassQL.: [evidence span withheld — anchor status: not_found]
- [methods] QUERY scaninfo(MS1DATA) FILTER MS1MZ=207.1418:TOLERANCEPPM=2.5 AND RTMIN=1.0 AND RTMAX=1.2: "QUERY scaninfo(MS1DATA) FILTER MS1MZ=207.1418:TOLERANCEPPM=2.5 AND RTMIN=1.0 AND RTMAX=1.2"
- [readme] Requires massqlab_config. to be configured upfront: [evidence span withheld — anchor status: not_found]
- [results] Statistical assessment of retention time yielded RSD in all SST analytes ranging from 0.5% to 2.3%: "Statistical assessment of retention time, Table S2, yielded a relative standard deviation (RSD) in all SST analytes ranging from 0.5% to 2.3%"
