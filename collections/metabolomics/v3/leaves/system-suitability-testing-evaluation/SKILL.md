---
name: system-suitability-testing-evaluation
description: Use when a UHPLC-HRMS (or similar) sequence needs pre-acquisition performance
  checks — e.g. six deuterated acylcarnitine SST standards in mzML format — to verify
  retention times, EICs, and MS/MS product ions before committing to full data collection.
when_to_use_negative:
- Your goal includes evaluating peak intensity and peak shape — the paper explicitly
  states 'the intensity and peak shape were not evaluated'
- You need consistent MS/MS-based SST for DDA acquisitions — 'the stochastic nature
  of MS/MS triggering using DDA is problematic for assessing consistency using SST'
- Input is not mass spectrometry data (or cannot be converted to mzML); MassQL queries
  operate on scaninfo(MS1DATA)/scaninfo(MS2DATA) from mzML files
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_3520
tools:
- name: MassQLab
  role: Python/Jupyter implementation of MassQL that applies named queries to a directory
    of mzML files, tabulates peak areas, and exports CSV and PDF outputs
  repo: https://github.com/JohnsonDylan/MassQLab
- name: MassQL
  role: Domain-specific query language used to write the MS1/MS2 suitability queries
    (mz, ppm tolerance, RT windows, product ions, neutral losses)
  repo: https://github.com/mwang87/MassQueryLanguage
- name: MSConvert
  role: Converts vendor .raw files to mzML with default settings prior to MassQL querying;
    MassQLab's built-in convert_raw is experimental
  repo: https://proteowizard.sourceforge.io/
provenance:
  source_task_ids:
  - task_001
  - task_007
  - task_003
  source_papers:
  - doi: 10.1002/rcm.10132
    title: 'Open‐Source System Suitability: Mass Spectrometry Query Language Lab (MassQLab)'
  source_content_hash: sha256:e3b4fd789e084e732aff5e204612c3b4d789f416e87a8b6f6cb83b06e4240b67
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/JohnsonDylan/MassQLab
  tools:
  - MSConvert
  - MassQL
  - MassQLab
  - Python
  - matplotlib
  - openpyxl
  - reportlab
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 14
---

# system-suitability-testing-evaluation

## Summary

Evaluate LC-MS/MS system suitability by running named MassQL queries (EICs, precursor/product-ion filters, neutral-loss checks) over a directory of mzML files using MassQLab, tabulating extracted peak areas and rendering PDF summaries. Addresses the fact that only ~2% of surveyed papers performed SST.

## When to use

Use when a UHPLC-HRMS (or similar) sequence needs pre-acquisition performance checks — e.g. six deuterated acylcarnitine SST standards in mzML format — to verify retention times, EICs, and MS/MS product ions before committing to full data collection.

## When NOT to use

- Your goal includes evaluating peak intensity and peak shape — the paper explicitly states 'the intensity and peak shape were not evaluated'
- You need consistent MS/MS-based SST for DDA acquisitions — 'the stochastic nature of MS/MS triggering using DDA is problematic for assessing consistency using SST'
- Input is not mass spectrometry data (or cannot be converted to mzML); MassQL queries operate on scaninfo(MS1DATA)/scaninfo(MS2DATA) from mzML files

## Inputs

- Directory of mass spectrometry data in mzML format (e.g. SST mixture of six deuterated acylcarnitine standards; MassIVE MSV000098825)
- Vendor .raw files plus ProteoWizard MSConvert for conversion (optional; MassQLab convert_raw is experimental)
- Query file (JSON, CSV, or XLSX) with 'name' and 'query' fields defining MassQL queries per MS1/MS2 type
- massqlab_config. specifying data_directory, queryfile, output_directory, analysis, and cache_setting

## Outputs

- ms1_raw_df.csv, ms1_analysis_df.csv, ms1_RT_analysis_df.csv — query results, Gaussian peak fits, and RT statistics
- ms1_traces.pdf, ms1_summary_areas.pdf, ms1_summary_traces.pdf, ms1_consolidated_traces.png — EIC plots and peak areas per query/file
- ms2_raw_df.csv, ms2_analysis_df.csv — MS2 scan/product-ion intensity results
- ms2_plots.pdf, ms2_summary_plots.pdf, ms2_cluster_plots.pdf — MS2 product-ion spectra summaries
- Results organized by timestamp under MassQLab_Output in the output_directory

## How to apply

Convert vendor .raw files to mzML with MSConvert (default settings), since MassQL requires the open mzML format. Compose a query file (JSON, CSV, or XLSX) with a unique 'name' and 'query' per check, e.g. MS1 EIC queries like 'QUERY scaninfo(MS1DATA) FILTER MS1MZ=207.1418:TOLERANCEPPM=2.5 AND RTMIN=1.0 AND RTMAX=1.2' and MS2 checks such as 'WHERE MS2PREC=429.3765:TOLERANCEPPM=2.5 ... FILTER MS2PROD=85.0281:TOLERANCEPPM=10' and 'WHERE MS2NL=176.0321 AND MS2PROD=85.02915'. Configure massqlab_config. with data_directory and queryfile, then run the console workflow so each query is applied to every mzML file. MassQLab tabulates extracted chromatographic peak areas and Gaussian fits, exporting CSVs (ms1_raw_df.csv, ms1_analysis_df.csv, ms2_raw_df.csv) and PDF summaries (ms1_summary_areas.pdf, ms2_summary_plots.pdf). Flag anomalies by inspecting RT/EIC plots across batches — e.g. column-lot retention-time shifts or a BEH Amide column run with the wrong (non-F5) method — and judge success by RT RSD of 0.5%–2.3% across SST analytes and peaks meeting detection criteria.

## Related tools

- **MassQLab** (Python/Jupyter implementation of MassQL that applies named queries to a directory of mzML files, tabulates peak areas, and exports CSV and PDF outputs) — https://github.com/JohnsonDylan/MassQLab
- **MassQL** (Domain-specific query language used to write the MS1/MS2 suitability queries (mz, ppm tolerance, RT windows, product ions, neutral losses)) — https://github.com/mwang87/MassQueryLanguage
- **MSConvert** (Converts vendor .raw files to mzML with default settings prior to MassQL querying; MassQLab's built-in convert_raw is experimental) — https://proteowizard.sourceforge.io/

## Examples

```
py src\MassQLab_console.py
```

## Evaluation signals

- Every detected SST peak meets MassQLab's detection criteria to be considered valid in downstream analysis
- Retention time RSD across all six deuterated acylcarnitine SST analytes falls in the reported 0.5%–2.3% range over ~6 months, two instrument platforms, and five operators
- ms1_raw_df.csv / ms1_analysis_df.csv exist with Gaussian peak fits per query and file; peak areas tabulated for each named query
- PDF outputs (e.g. ms1_summary_areas.pdf, ms2_cluster_plots.pdf) render plots for each named MassQL query, and EIC patterns are consistent with the F5 chromatographic method (no BEH Amide/wrong-method signatures or batch-clustered column-lot RT shifts)
- CID vs HCD product-ion (m/z 85.0824, TOLERANCEPPM=10) comparisons increase logarithmically with NCE 10–60, with CID signal lower than HCD at every NCE step

## Limitations

- Intensity and peak shape were not evaluated in the illustrative SST assessment
- Stochastic DDA MS/MS triggering is problematic for assessing consistency via SST, though normalization to total signal helped
- MassQLab's raw-to-mzML conversion (convert_raw) is experimental; converting via ProteoWizard MSConvert is recommended
- SST MS/MS evaluation was judged not warranted in the authors' use case, and MassQLab's performance/functions are still to be refined and expanded

## Evidence

- [intro] MassQLab applies a series of queries (written in the language of MassQL) to a directory containing mass spectrometry data in mzML format. Results are tabulated and saved as images and csv files.: "MassQLab applies a series of queries (written in the language of MassQL) to a directory containing mass spectrometry data in mzML format. Results are tabulated and saved as images and csv files."
- [methods] QUERY scaninfo(MS1DATA) FILTER MS1MZ=207.1418:TOLERANCEPPM=2.5 AND RTMIN=1.0 AND RTMAX=1.2: "QUERY scaninfo(MS1DATA) FILTER MS1MZ=207.1418:TOLERANCEPPM=2.5 AND RTMIN=1.0 AND RTMAX=1.2"
- [results] Statistical assessment of retention time, Table S2, yielded a relative standard deviation (RSD) in all SST analytes ranging from 0.5% to 2.3%: "Statistical assessment of retention time, Table S2, yielded a relative standard deviation (RSD) in all SST analytes ranging from 0.5% to 2.3%"
- [methods] Raw data files (.raw) were converted to mzML format using MSConvert (https://proteowizard.sourceforge.io/) with default settings.: "Raw data files (.raw) were converted to mzML format using MSConvert (https://proteowizard.sourceforge.io/) with default settings."
- [results] Note, the intensity and peak shape were not evaluated.: "Note, the intensity and peak shape were not evaluated."
- [readme] Each query in the file must include the following: name: A unique name for each query (per MS1/MS2 type).: "Each query in the file must include the following: **`name`**: A unique name for each query (per MS1/MS2 type)."
- [discussion] the stochastic nature of MS/MS triggering using DDA is problematic for assessing consistency using SST, albeit normalization to the total signal helped in our evaluation.: "the stochastic nature of MS/MS triggering using DDA is problematic for assessing consistency using SST, albeit normalization to the total signal helped in our evaluation."
