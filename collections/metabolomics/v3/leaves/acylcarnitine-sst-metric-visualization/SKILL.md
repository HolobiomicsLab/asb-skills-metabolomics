---
name: acylcarnitine-sst-metric-visualization
description: Use when MassQL queries (e.g., MS1MZ=207.1418:TOLERANCEPPM=2.5 or MS2PREC=429.3765
  filters) have been run over a directory of mzML SST files and the results must be
  tabulated and rendered as EIC/Gaussian/peak-area plots and PDF summaries.
when_to_use_negative:
- Input data is still in vendor-specific .raw format and has not been converted to
  open mzML — convert with ProteoWizard MSConvert first
- The goal is a one-off query of a single spectrum rather than tabulated, visualized
  results across a series of files (use the plain MassQL CLI instead)
- Intensity and peak shape must be evaluated as SST metrics, which MassQLab's illustrative
  acylcarnitine assessment explicitly did not evaluate
edam_operation: http://edamontology.org/operation_3204
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: MassQLab
  role: Python Jupyter notebook implementation that applies MassQL queries, tabulates,
    and visualizes SST results
  repo: https://github.com/JohnsonDylan/MassQLab
- name: MassQL
  role: Query language used to extract EICs and MS/MS product ion spectra from mzML
    files
  repo: https://github.com/mwang87/MassQueryLanguage
- name: Python
  role: Language (3.9) in which MassQLab is developed and run
- name: MSConvert
  role: Converts vendor .raw files to mzML required before querying and visualization
  repo: https://proteowizard.sourceforge.io/
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1002/rcm.10132
    title: 'Open‐Source System Suitability: Mass Spectrometry Query Language Lab (MassQLab)'
  source_content_hash: sha256:8e46be0dfc03b5019590f378d9098a6cec3213255a43727af8519c5e92063664
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/JohnsonDylan/MassQLab
  tools:
  - JupyterLab
  - MSConvert
  - MassQL
  - MassQLab
  - Python
  - matplotlib
  - openpyxl
  - reportlab
  edam_operation: http://edamontology.org/operation_3204
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7647058823529411
n_claims: 17
---

# acylcarnitine-ssT-metric-visualization

## Summary

Visualize MassQLab system suitability testing (SST) results for deuterated acylcarnitine standards by plotting extracted ion chromatograms, Gaussian peak fits, integrated peak areas, and MS/MS product ion spectra from tabulated MassQL query results, organized per query/file in a MassQLab_Output directory.

## When to use

Use when MassQL queries (e.g., MS1MZ=207.1418:TOLERANCEPPM=2.5 or MS2PREC=429.3765 filters) have been run over a directory of mzML SST files and the results must be tabulated and rendered as EIC/Gaussian/peak-area plots and PDF summaries.

## When NOT to use

- Input data is still in vendor-specific .raw format and has not been converted to open mzML — convert with ProteoWizard MSConvert first
- The goal is a one-off query of a single spectrum rather than tabulated, visualized results across a series of files (use the plain MassQL CLI instead)
- Intensity and peak shape must be evaluated as SST metrics, which MassQLab's illustrative acylcarnitine assessment explicitly did not evaluate

## Inputs

- Directory of mzML mass spectrometry files (SST mixture of deuterated acylcarnitine standards acquired by UHPLC-HRMS)
- Query file (JSON, CSV, or XLSX) with named MassQL queries (name + query fields)
- massqlab_config. with data_directory, queryfile, output_directory, analysis, cache_setting options

## Outputs

- ms1_raw_df.csv and ms1_analysis_df.csv (raw query results, Gaussian peak fitting analysis)
- ms1_traces.pdf, ms1_summary_traces.pdf, ms1_summary_traces_inverse.pdf (Gaussian fit plots per query/file)
- ms1_summary_areas.pdf and ms1_summary_areas_inverse.pdf (peak areas per query/per file)
- ms2_raw_df.csv, ms2_analysis_df.csv, ms2_plots.pdf, ms2_summary_plots.pdf, ms2_cluster_plots.pdf
- MassQLab_Output directory organized by timestamp inside the output_directory

## How to apply

Configure massqlab_config. with the data_directory of mzML files, a queryfile (JSON/CSV/XLSX) giving each MassQL query a unique name, and output_directory; then run an entry point (e.g., MassQLab_console.py or MassQLab_notebook.ipynb) so MassQLab applies the queries to the directory, tabulates results, and saves them as images and csv files. For MS1, the workflow fits Gaussian curves to extracted ion chromatograms (e.g., m/z 207.1418 at 2.5 ppm within RT 1.0–1.2 min) and reports ms1_traces.pdf, ms1_summary_traces.pdf, and ms1_summary_areas.pdf alongside ms1_raw_df.csv and ms1_analysis_df.csv. For MS2, scan intensities around precursor 429.3765 (2.5 ppm, RT 9.0–9.5 min) and product ions such as 85.0281 (10 ppm tolerance) are visualized in ms2_plots.pdf and clustered summaries. Plot summaries are produced in two directions — per query across files and inverse (per file across queries) — to expose retention-time and peak-area consistency across the ~6 months, two instrument platforms, and five operators. Set analysis=true so downstream peak-fitting/peak-picking analysis runs, and optionally enable cache_setting to store an intermediate Feather file for faster re-querying.

## Related tools

- **MassQLab** (Python Jupyter notebook implementation that applies MassQL queries, tabulates, and visualizes SST results) — https://github.com/JohnsonDylan/MassQLab
- **MassQL** (Query language used to extract EICs and MS/MS product ion spectra from mzML files) — https://github.com/mwang87/MassQueryLanguage
- **Python** (Language (3.9) in which MassQLab is developed and run)
- **MSConvert** (Converts vendor .raw files to mzML required before querying and visualization) — https://proteowizard.sourceforge.io/

## Examples

```
py src\MassQLab_console.py
```

## Evaluation signals

- MassQLab_Output directory exists inside output_directory (or data_directory by default) and contains timestamp-organized result folders
- ms1_traces.pdf contains a Gaussian-fit EIC plot for every named query per mzML file, and ms1_summary_traces_inverse.pdf summarizes per file
- ms1_raw_df.csv and ms1_analysis_df.csv are present and contain raw query results and peak-fitting analysis for every query/file combination
- Retention-time RSD values for the deuterated acylcarnitine SST analytes fall in the reported 0.5%–2.3% range
- MS2 queries (e.g., MS2PROD=85.0281:TOLERANCEPPM=10) yield ms2_plots.pdf with scan intensities and ms2_summary_plots.pdf with the top scan per query

## Limitations

- Intensity and peak shape were not evaluated in the illustrative SST assessment
- The stochastic nature of DDA MS/MS triggering is problematic for assessing consistency using SST, although normalization to total signal helped
- convert_raw raw-to-mzML conversion is experimental; ProteoWizard msconvert is recommended, and the GUI may continue running in the background if closed during processing

## Evidence

- [intro] MassQLab applies a series of queries (written in the language of MassQL) to a directory containing mass spectrometry data in mzML format. Results are tabulated and saved as images and csv files.: "MassQLab applies a series of queries (written in the language of MassQL) to a directory containing mass spectrometry data in mzML format. Results are tabulated and saved as images and csv files."
- [readme] Results are saved in MassQLab_Output inside the defined output_directory (data_directory is used by default), organized by timestamp.: "Results are saved in `MassQLab_Output` inside the defined `output_directory` (`data_directory` is used by default), organized by timestamp."
- [methods] MassQLab uses MassQL to apply a series of queries to a series of files and tabulates and visualizes the results.: "MassQLab uses MassQL to apply a series of queries to a series of files and tabulates and visualizes the results."
- [results] Statistical assessment of retention time, Table S2, yielded a relative standard deviation (RSD) in all SST analytes ranging from 0.5% to 2.3%: "Statistical assessment of retention time, Table S2, yielded a relative standard deviation (RSD) in all SST analytes ranging from 0.5% to 2.3%"
- [results] Note, the intensity and peak shape were not evaluated.: "Note, the intensity and peak shape were not evaluated."
