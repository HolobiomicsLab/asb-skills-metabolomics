---
name: lcms-system-suitability-assessment
description: Use when evaluating UHPLC-HRMS acquisition performance prior to data
  collection — e.g., assessing RT stability, peak areas, or MS/MS product ion consistency
  of an SST standard mixture acquired over months by multiple operators.
when_to_use_negative:
- Input data are already feature tables or processed peak areas rather than mzML acquisitions
  — MassQLab queries raw mzML spectra
- The goal is rigorous quantitative MS/MS evaluation in DDA mode — the stochastic
  nature of DDA MS/MS triggering is problematic for assessing consistency using SST
- Vendor-specific raw files (.raw) without conversion — mzML conversion is a prerequisite
  before MassQL use
edam_operation: http://edamontology.org/operation_3214
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: MassQLab
  role: Python/Jupyter implementation that applies a series of named MassQL queries
    to a directory of mzML files and tabulates/visualizes results (images and csv
    files) for SST
  repo: https://github.com/JohnsonDylan/MassQLab
- name: MassQL
  role: Query language used to express precursor (MS2PREC), product ion (MS2PROD),
    and neutral loss (MS2NL) filters with ppm tolerances applied per file
  repo: https://github.com/mwang87/MassQueryLanguage
- name: MSConvert
  role: Converts vendor .raw files to mzML format with default settings, a prerequisite
    before MassQL/MassQLab use
  repo: https://proteowizard.sourceforge.io/
- name: Python
  role: Language in which MassQL and MassQLab are implemented (MassQLab developed
    using Python 3.9)
  repo: https://github.com/JohnsonDylan/MassQLab
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1002/rcm.10132
    title: 'Open‐Source System Suitability: Mass Spectrometry Query Language Lab (MassQLab)'
  source_content_hash: sha256:fc0f8e023456763131d97ac234523052b206e97e24567f92e3961dee8364b852
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/JohnsonDylan/MassQLab
  tools:
  - MSConvert
  - MassQL
  - MassQLab
  - Python
  edam_operation: http://edamontology.org/operation_3214
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# lcms-system-suitability-assessment

## Summary

Performs LC-MS/MS system suitability testing (SST) by applying named MassQL queries via MassQLab to directories of mzML files, tracking retention time RSD, extracted ion chromatograms, and MS/MS product ion reproducibility of a deuterated acylcarnitine SST mixture across days, platforms, and operators.

## When to use

Use when evaluating UHPLC-HRMS acquisition performance prior to data collection — e.g., assessing RT stability, peak areas, or MS/MS product ion consistency of an SST standard mixture acquired over months by multiple operators.

## When NOT to use

- Input data are already feature tables or processed peak areas rather than mzML acquisitions — MassQLab queries raw mzML spectra
- The goal is rigorous quantitative MS/MS evaluation in DDA mode — the stochastic nature of DDA MS/MS triggering is problematic for assessing consistency using SST
- Vendor-specific raw files (.raw) without conversion — mzML conversion is a prerequisite before MassQL use

## Inputs

- Directory of LC-MS/MS mzML files (vendor raw files converted with MSConvert default settings)
- SST mixture of deuterated acylcarnitine standards (1 ug/mL, e.g. from MassIVE MSV000098825 deposit)
- MassQL query file (JSON, CSV, or XLSX) with 'name' and 'query' fields per query
- massqlab_config. specifying data_directory, queryfile, and optional output_directory, analysis, cache_setting, convert_raw, msconvertexe

## Outputs

- ms1_raw_df.csv, ms1_analysis_df.csv, ms1_RT_analysis_df.csv with query results and RT statistics
- ms1_traces.pdf and ms1_summary_areas.pdf with Gaussian peak fits and peak areas per query/file
- ms2_raw_df.csv, ms2_analysis_df.csv with MS/MS scan and product ion intensities
- ms2_plots.pdf, ms2_summary_plots.pdf, ms2_cluster_plots.pdf visualizing MS/MS scan intensities and energy-level clustering
- Normalized product ion relative intensities and relative standard deviations across the SST acquisition series

## How to apply

Acquire the SST mixture (1 ug/mL of six deuterated acylcarnitine standards) in positive ionization mode and convert raw files to mzML with MSConvert default settings. Write a query file (JSON, CSV, or XLSX) of named MassQL queries — e.g. 'QUERY scaninfo(MS2DATA) WHERE MS2PREC=429.3765:TOLERANCEPPM=2.5 AND RTMIN=9.0 AND RTMAX=9.5' for DDA MS/MS scans, adding 'FILTER MS2PROD=85.0281:TOLERANCEPPM=10' to extract diagnostic carnitine fragment intensities. Configure massqlab_config. with the data directory and queryfile, then run MassQLab (console, GUI, or notebook) so each query is applied to each mzML file and results are tabulated. Assess retention time RSD (expected 0.5%–2.3% across ~6 months, two platforms, five operators), peak areas via Gaussian fits, and MS/MS product ion relative intensity after normalizing each product ion intensity to the total signal of its scan. The validation check is qualitative: product ions and their relative intensities should be reproducible across days, but the stochastic nature of DDA MS/MS triggering means SST MS/MS evaluation may not be warranted for this use case.

## Related tools

- **MassQLab** (Python/Jupyter implementation that applies a series of named MassQL queries to a directory of mzML files and tabulates/visualizes results (images and csv files) for SST) — https://github.com/JohnsonDylan/MassQLab
- **MassQL** (Query language used to express precursor (MS2PREC), product ion (MS2PROD), and neutral loss (MS2NL) filters with ppm tolerances applied per file) — https://github.com/mwang87/MassQueryLanguage
- **MSConvert** (Converts vendor .raw files to mzML format with default settings, a prerequisite before MassQL/MassQLab use) — https://proteowizard.sourceforge.io/
- **Python** (Language in which MassQL and MassQLab are implemented (MassQLab developed using Python 3.9)) — https://github.com/JohnsonDylan/MassQLab

## Examples

```
py src\MassQLab_console.py  # after configuring massqlab_config. with data_directory and queryfile containing e.g. QUERY scaninfo(MS2DATA) WHERE MS2PREC=429.3765:TOLERANCEPPM=2.5 AND RTMIN=9.0 AND RTMAX=9.5 FILTER MS2PROD=85.0281:TOLERANCEPPM=10
```

## Evaluation signals

- Retention time RSD across all SST analytes should fall roughly in the 0.5%–2.3% range over ~6 months, two platforms, and five operators
- After normalizing product ion intensity to the total signal of its scan, product ions and their relative intensities should show consistent trends across days (e.g. m/z 85.0824 increasing logarithmically for both CID and HCD)
- Output directory MassQLab_Output contains the expected files (ms1_raw_df.csv, ms2_raw_df.csv, summary PDFs) organized by timestamp
- Every query in the query file returns results for each mzML file, with peaks meeting MassQLab detection criteria counted as valid
- Precursor filters match at 2.5 ppm tolerance and diagnostic product ions (e.g. MS2PROD=85.0281) at 10 ppm tolerance under 30000 resolution and 1.5 m/z isolation

## Limitations

- SST MS/MS evaluation is not warranted in the authors' use case because the stochastic nature of MS/MS triggering using DDA is problematic for assessing consistency using SST, albeit normalization to the total signal helped in the evaluation
- MassQLab currently processes all files in the data_directory and its convert_raw feature is experimental; converting files with ProteoWizard msconvert beforehand is recommended
- MassQLab performance and available functions are still to be refined/expanded, and no software version number is provided
- Intensity and peak shape were not evaluated in the illustrative SST assessment

## Evidence

- [methods] MassQLab is a Python-based Jupyter notebook implementation of MassQL.: [evidence span withheld — anchor status: not_found]
- [methods] Raw data files (.raw) were converted to mzML format using MSConvert (https://proteowizard.sourceforge.io/) with default settings.: "Raw data files (.raw) were converted to mzML format using MSConvert (https://proteowizard.sourceforge.io/) with default settings."
- [results] Statistical assessment of retention time, Table S2, yielded a relative standard deviation (RSD) in all SST analytes ranging from 0.5% to 2.3%: "Statistical assessment of retention time, Table S2, yielded a relative standard deviation (RSD) in all SST analytes ranging from 0.5% to 2.3%"
- [discussion] the stochastic nature of MS/MS triggering using DDA is problematic for assessing consistency using SST, albeit normalization to the total signal helped in our evaluation.: "the stochastic nature of MS/MS triggering using DDA is problematic for assessing consistency using SST, albeit normalization to the total signal helped in our evaluation."
- [readme] MassQLab applies a series of queries (written in the language of MassQL) to a directory containing mass spectrometry data in mzML format. Results are tabulated and saved as images and csv files.: "MassQLab applies a series of queries (written in the language of MassQL) to a directory containing mass spectrometry data in mzML format. Results are tabulated and saved as images and csv files."
- [readme] Get MS2 scans with a precursor ion matching m/z 429.3765, retention time between 9.0 and 9.5 minutes, and return intensity of the peak with m/z 85.0281: "Get MS2 scans with a precursor ion matching m/z 429.3765, retention time between 9.0 and 9.5 minutes, and return intensity of the peak with m/z 85.0281"
