---
name: chromatogram-plotting-from-massql-results
description: Use when MassQL queries have been run over a directory of mzML files
  (e.g., SST deuterated acylcarnitine standards on a UHPLC-HRMS) and the peak/scan
  results need to be visualized as EICs, Gaussian fits, and MS2 plots and exported
  as tables and PDF reports.
when_to_use_negative:
- Input data is not in an open-source MS format (mzML) — vendor .raw files must first
  be converted via MSConvert/msconvert before MassQL queries and plotting can run
- The goal is evaluating absolute intensity or peak shape quality — the illustrative
  SST assessment explicitly notes 'the intensity and peak shape were not evaluated'
- You need interactive spectral exploration rather than static per-query reports —
  MassQL itself offers Dash-based visualization and a web /visualize/ms1 and /visualize/ms2
  API instead
edam_operation: http://edamontology.org/operation_3203
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: MassQLab
  role: Python/Jupyter implementation that applies the MassQL queries, tabulates results,
    and generates the EIC/Gaussian-fit/MS2 plot images and csv tables
  repo: https://github.com/JohnsonDylan/MassQLab
- name: MassQL
  role: Domain-specific query language whose results (per scan intensities, EICs,
    product ions) are what get plotted and tabulated
  repo: https://github.com/mwang87/MassQueryLanguage
- name: Python
  role: Language in which MassQLab is implemented (Python 3.9); plotting/export extensions
    use matplotlib, openpyxl, and reportlab
- name: MSConvert
  role: Converts vendor .raw files to mzML, the required input format for querying
    and plotting
  repo: https://proteowizard.sourceforge.io/
- name: JupyterLab
  role: Notebook entry points (MassQLab_notebook.ipynb, MassQLab_console.ipynb, MassQLab_GUI.ipynb)
    that walk through or launch the plotting workflow
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1002/rcm.10132
    title: 'Open‐Source System Suitability: Mass Spectrometry Query Language Lab (MassQLab)'
  source_content_hash: sha256:79e8652f66aa231775682d2cfea6e3e9414653c923db47e7289669c45eca7c59
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
  edam_operation: http://edamontology.org/operation_3203
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6842105263157895
n_claims: 19
---

# chromatogram-plotting-from-massql-results

## Summary

Visualizes and tabulates MassQL query results from MassQLab system suitability testing by plotting extracted ion chromatograms, Gaussian peak fits, peak areas, and MS/MS product ion spectra as PDF/PNG images, and exporting raw and analysis csv tables, all organized by timestamp in a MassQLab_Output directory.

## When to use

Use when MassQL queries have been run over a directory of mzML files (e.g., SST deuterated acylcarnitine standards on a UHPLC-HRMS) and the peak/scan results need to be visualized as EICs, Gaussian fits, and MS2 plots and exported as tables and PDF reports.

## When NOT to use

- Input data is not in an open-source MS format (mzML) — vendor .raw files must first be converted via MSConvert/msconvert before MassQL queries and plotting can run
- The goal is evaluating absolute intensity or peak shape quality — the illustrative SST assessment explicitly notes 'the intensity and peak shape were not evaluated'
- You need interactive spectral exploration rather than static per-query reports — MassQL itself offers Dash-based visualization and a web /visualize/ms1 and /visualize/ms2 API instead

## Inputs

- Directory of mzML files (converted from vendor .raw via MSConvert)
- Query file (JSON, CSV, or XLSX) with 'name' and 'query' fields defining MassQL queries
- massqlab_config. specifying data_directory, queryfile, output_directory, and analysis flags
- Optional cached Feather intermediate of prior query results (cache_setting=true)

## Outputs

- ms1_raw_df.csv and ms1_analysis_df.csv (raw query results and Gaussian peak-fitting analysis)
- ms1_traces.pdf, ms1_summary_traces.pdf, ms1_summary_traces_inverse.pdf (Gaussian fit plots per query/file)
- ms1_summary_areas.pdf and ms1_summary_areas_inverse.pdf (peak areas per query/file)
- ms1_consolidated_traces.png (consolidated image of all ms1 traces with Gaussian fit)
- ms2_raw_df.csv and ms2_analysis_df.csv (raw MS2 results and peak-picking analysis)
- ms2_plots.pdf, ms2_summary_plots.pdf, ms2_cluster_plots.pdf, ms2_cluster_plots_group.pdf (MS2 scan intensity and clustered plots)
- Optionally a compiled .xlsx of tabulated peak areas and a resultant PDF report per named query

## How to apply

Run MassQLab (console, GUI, or notebook entry point) with a massqlab_config. pointing at the mzML data directory and a query file (JSON, CSV, or XLSX) whose entries carry a unique 'name' and the MassQL 'query' string, e.g., QUERY scaninfo(MS1DATA) FILTER MS1MZ=207.1418:TOLERANCEPPM=2.5 AND RTMIN=1.0 AND RTMAX=1.2. With 'analysis': true, the pipeline tabulates raw query results and metadata (ms1_raw_df.csv, ms2_raw_df.csv), performs Gaussian peak fitting (MS1) or peak picking (MS2), and renders per-query and summary plots — ms1_traces.pdf with all Gaussian fits per query/file, ms1_summary_areas.pdf of peak areas, and ms2_plots.pdf of scan intensities — into a timestamped MassQLab_Output folder. Plotting is done in Python (matplotlib), and the workflow can be extended to export the tabulated peak areas as .xlsx (openpyxl) and compile per-query plots into a single PDF report (reportlab). Judge success by checking that every named query and mzML file combination appears in both the tabular outputs and the plot sections.

## Related tools

- **MassQLab** (Python/Jupyter implementation that applies the MassQL queries, tabulates results, and generates the EIC/Gaussian-fit/MS2 plot images and csv tables) — https://github.com/JohnsonDylan/MassQLab
- **MassQL** (Domain-specific query language whose results (per scan intensities, EICs, product ions) are what get plotted and tabulated) — https://github.com/mwang87/MassQueryLanguage
- **Python** (Language in which MassQLab is implemented (Python 3.9); plotting/export extensions use matplotlib, openpyxl, and reportlab)
- **MSConvert** (Converts vendor .raw files to mzML, the required input format for querying and plotting) — https://proteowizard.sourceforge.io/
- **JupyterLab** (Notebook entry points (MassQLab_notebook.ipynb, MassQLab_console.ipynb, MassQLab_GUI.ipynb) that walk through or launch the plotting workflow)

## Examples

```
py src\MassQLab_console.py
```

## Evaluation signals

- MassQLab_Output directory exists inside the defined output_directory (or data_directory by default), organized by timestamp
- ms1_raw_df.csv, ms1_analysis_df.csv, ms2_raw_df.csv, and ms2_analysis_df.csv are present and populated for every query/file combination
- ms1_traces.pdf contains a Gaussian-fit trace section for each named query per mzML file, and ms1_summary_areas.pdf shows peak areas for every query
- ms2_plots.pdf renders scan intensities for each MS2 query with lines connecting shared MS1 scans
- For an optional .xlsx/PDF report export: the xlsx contains tabulated peak areas for every query-file combination and the PDF has a plot section per named query

## Limitations

- The illustrative SST assessment did not evaluate intensity and peak shape ('Note, the intensity and peak shape were not evaluated.')
- MS/MS SST evaluation may be unwarranted for some use cases because of the stochastic nature of DDA MS/MS triggering, though normalization to total signal helped
- Standalone GUI and some advanced functions are still in development; the authors plan to refine MassQLab performance and expand available functions, and no software version number is stated
- The convert_raw config option is experimental; ProteoWizard msconvert is recommended for raw-to-mzML conversion instead
- Query names should avoid being very long to prevent output issues

## Evidence

- [intro] MassQLab applies a series of queries (written in the language of MassQL) to a directory containing mass spectrometry data in mzML format. Results are tabulated and saved as images and csv files.: "MassQLab applies a series of queries (written in the language of MassQL) to a directory containing mass spectrometry data in mzML format. Results are tabulated and saved as images and csv files."
- [methods] MassQLab uses MassQL to apply a series of queries to a series of files and tabulates and visualizes the results.: "MassQLab uses MassQL to apply a series of queries to a series of files and tabulates and visualizes the results."
- [readme] Results are saved in `MassQLab_Output` inside the defined `output_directory` (`data_directory` is used by default), organized by timestamp.: "Results are saved in `MassQLab_Output` inside the defined `output_directory` (`data_directory` is used by default), organized by timestamp."
- [readme] ms1_traces.pdf: All Gaussian fits per query/file: "ms1_traces.pdf: All Gaussian fits per query/file"
