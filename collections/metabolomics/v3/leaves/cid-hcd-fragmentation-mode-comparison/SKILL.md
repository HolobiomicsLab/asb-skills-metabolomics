---
name: cid-hcd-fragmentation-mode-comparison
description: Use when LC-MS/MS DDA data were acquired on a system suitability standard
  under both HCD and CID at stepped normalized collision energies (e.g., NCE 10-60),
  and the goal is to compare product ion signal between the two fragmentation modes.
when_to_use_negative:
- Input is vendor .raw data that has not been converted to mzML - MassQL requires
  an open-source MS file format first (MassQLab's built-in convert_raw is experimental)
- The goal is general SST of retention time or extracted ion chromatograms rather
  than MS/MS product ion comparison across fragmentation modes and collision energies
- 'Data are not DDA MS/MS: the article notes the stochastic nature of MS/MS triggering
  in DDA is problematic for assessing consistency, so untargeted or non-DDA acquisitions
  are a poor fit'
edam_operation: http://edamontology.org/operation_3214
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0219
tools:
- name: MassQLab
  role: Python 3.9 Jupyter/console pipeline that applies MassQL queries to a directory
    of mzML files and tabulates/visualizes product ion signal results
  repo: https://github.com/JohnsonDylan/MassQLab
- name: MassQL
  role: Query language used to select MS2 scans by precursor (2.5 ppm) and extract
    the m/z 85.0824 product ion intensity (10 ppm)
  repo: https://github.com/mwang87/MassQueryLanguage
- name: MSConvert
  role: Converts vendor .raw files to mzML with default settings prior to MassQL querying
  repo: https://proteowizard.sourceforge.io/
- name: Python
  role: Runtime for MassQLab, openpyxl export, matplotlib plotting, and logarithmic
    fitting
- name: matplotlib
  role: Renders signal-versus-NCE plots for CID and HCD compiled into the report
- name: openpyxl
  role: Exports tabulated product ion peak areas as product_ion_signal_vs_nce.xlsx
- name: reportlab
  role: Compiles plots and results into the MassQLab PDF report
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1002/rcm.10132
    title: 'Open‐Source System Suitability: Mass Spectrometry Query Language Lab (MassQLab)'
  source_content_hash: sha256:903db2b851fbe44db55fa1f50917d5614b5b154b5a09e2f4d345aaf8b070350d
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
  edam_operation: http://edamontology.org/operation_3214
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0219
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# cid-hcd-fragmentation-mode-comparison

## Summary

A MassQLab/MassQL workflow that quantifies MS2 product ion signal (e.g., m/z 85.0824 of deuterated acylcarnitines) under both CID and HCD fragmentation across normalized collision energies 10-60, then fits signal-versus-NCE relationships and compares the two fragmentation modes.

## When to use

Use when LC-MS/MS DDA data were acquired on a system suitability standard under both HCD and CID at stepped normalized collision energies (e.g., NCE 10-60), and the goal is to compare product ion signal between the two fragmentation modes.

## When NOT to use

- Input is vendor .raw data that has not been converted to mzML - MassQL requires an open-source MS file format first (MassQLab's built-in convert_raw is experimental)
- The goal is general SST of retention time or extracted ion chromatograms rather than MS/MS product ion comparison across fragmentation modes and collision energies
- Data are not DDA MS/MS: the article notes the stochastic nature of MS/MS triggering in DDA is problematic for assessing consistency, so untargeted or non-DDA acquisitions are a poor fit

## Inputs

- LC-MS/MS DDA raw files (.raw) of a SST mixture acquired with both HCD and CID at NCE 10-60 (e.g., MassIVE MSV000098825)
- mzML-converted versions of the raw files (MSConvert, default settings)
- MassQL query file (JSON, CSV, or XLSX) with 'name' and 'query' fields per precursor/RT window/NCE/fragmentation-mode combination
- massqlab_config.json specifying data_directory and queryfile

## Outputs

- ms2_raw_df.csv / ms2_analysis_df.csv with m/z 85.0824 product ion peak areas per scan, NCE, and fragmentation mode
- ms2_plots.pdf, ms2_summary_plots.pdf, ms2_cluster_plots.pdf (clustered by energy level)
- Tabulated product ion peak areas vs NCE (e.g., product_ion_signal_vs_nce.xlsx via openpyxl)
- Signal-versus-NCE plots for CID and HCD with logarithmic fits, compiled into a PDF report (reportlab)

## How to apply

Download the SST mixture LC-MS/MS raw files (e.g., MassIVE MSV000098825) and convert them to mzML with MSConvert using default settings, since vendor-specific formats must be converted before MassQL use. Write MassQL MS2 queries of the form QUERY scaninfo(MS2DATA) WHERE MS2PREC=<precursor>:TOLERANCEPPM=2.5 AND RTMIN/RTMAX=<retention window> FILTER MS2PROD=85.0824:TOLERANCEPPM=10 into a query file (JSON, CSV, or XLSX) with a unique 'name' per query, and point massqlab_config.json at the mzML directory and the query file. Run MassQLab (Python 3.9) so each query is applied to each mzML file, tabulating the m/z 85.0824 product ion peak area for scans at NCE 10, 20, 30, 40, 50, and 60 under both HCD and CID (results land in MassQLab_Output as ms2_raw_df.csv and related PDFs). Export the tabulated peak areas (e.g., product_ion_signal_vs_nce.xlsx via openpyxl), render signal-versus-NCE plots for CID and HCD with matplotlib, and fit a logarithmic model of product ion signal versus NCE separately for each fragmentation mode. Verify the expected pattern: product ion signal increases logarithmically with NCE for both modes and CID signal is lower than HCD at every NCE step.

## Related tools

- **MassQLab** (Python 3.9 Jupyter/console pipeline that applies MassQL queries to a directory of mzML files and tabulates/visualizes product ion signal results) — https://github.com/JohnsonDylan/MassQLab
- **MassQL** (Query language used to select MS2 scans by precursor (2.5 ppm) and extract the m/z 85.0824 product ion intensity (10 ppm)) — https://github.com/mwang87/MassQueryLanguage
- **MSConvert** (Converts vendor .raw files to mzML with default settings prior to MassQL querying) — https://proteowizard.sourceforge.io/
- **Python** (Runtime for MassQLab, openpyxl export, matplotlib plotting, and logarithmic fitting)
- **matplotlib** (Renders signal-versus-NCE plots for CID and HCD compiled into the report)
- **openpyxl** (Exports tabulated product ion peak areas as product_ion_signal_vs_nce.xlsx)
- **reportlab** (Compiles plots and results into the MassQLab PDF report)

## Examples

```
results_df = msql_engine.process_query("QUERY scaninfo(MS2DATA) WHERE MS2PREC=429.3765:TOLERANCEPPM=2.5 AND RTMIN=9.0 AND RTMAX=9.5 FILTER MS2PROD=85.0281:TOLERANCEPPM=10", input_filename)
```

## Evaluation signals

- CID product ion signal at m/z 85.0824 is lower than HCD at every NCE step (10, 20, 30, 40, 50, 60)
- Product ion signal increases logarithmically with NCE for both CID and HCD (log model fits reasonably)
- Queries return scans within the specified RT window and precursor 2.5 ppm tolerance; product ion extracted at 10 ppm tolerance
- MassQLab_Output contains expected ms2 outputs (ms2_raw_df.csv, ms2_plots.pdf, ms2_cluster_plots.pdf) clustered by energy level
- Scans are correctly grouped by fragmentation mode and NCE step before fitting, since DDA triggering is stochastic

## Limitations

- The stochastic nature of MS/MS triggering using DDA is problematic for assessing consistency using SST, albeit normalization to the total signal helped in the evaluation
- MassQLab currently processes all files in the data_directory and convert_raw is experimental - convert with ProteoWizard msconvert first
- MassQL is tested in Python 3.9; other versions are not guaranteed to work
- The article's SST MS/MS evaluation was not warranted in their use case, and intensity/peak shape were not evaluated in the illustrative SST assessment

## Evidence

- [methods] article methods: "Raw data files (.raw) were converted to mzML format using MSConvert (https://proteowizard.sourceforge.io/) with default settings."
- [intro] article intro: "MassQLab applies a series of queries (written in the language of MassQL) to a directory containing mass spectrometry data in mzML format. Results are tabulated and saved as images and csv files."
- [discussion] article discussion: "the stochastic nature of MS/MS triggering using DDA is problematic for assessing consistency using SST, albeit normalization to the total signal helped in our evaluation."
- [readme] readme: "Results are saved in `MassQLab_Output` inside the defined `output_directory` (`data_directory` is used by default), organized by timestamp."
