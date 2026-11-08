---
name: collision-energy-response-curve-fitting
description: Use when DDA LC-MS/MS mzML files contain scans of the same precursor
  at multiple NCE steps and under both CID and HCD, and you need to compare product
  ion signal (e.g., m/z 85.0824, 10 ppm) versus NCE between fragmentation modes.
when_to_use_negative:
- Input is already a feature table or XIC peak list with no MS2 scans — MassQL requires
  mzML spectral data
- The goal is peak intensity and peak shape assessment of the MS1 SST analytes — the
  article explicitly did not evaluate intensity and peak shape in the MS/MS assessment
- Only a single NCE or a single fragmentation mode is available — the CID-vs-HCD log-fit
  comparison requires matched scans across NCE 10-60 in both modes
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: MassQLab
  role: Python 3.9 Jupyter/console implementation of MassQL that applies the MS2 product-ion
    queries to a directory of mzML files and tabulates/visualizes results
  repo: https://github.com/JohnsonDylan/MassQLab
- name: MassQL
  role: Query language used to select MS2 scans by precursor (2.5 ppm), RT window,
    and product ion m/z 85.0824 (10 ppm) for each NCE step
  repo: https://github.com/mwang87/MassQueryLanguage
- name: MSConvert
  role: Converts vendor .raw SST files to mzML (default settings) prior to MassQL
    querying
  repo: https://proteowizard.sourceforge.io/
- name: Python
  role: Runtime for MassQLab, logarithmic NCE fits, and result tabulation (openpyxl,
    matplotlib, reportlab)
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1002/rcm.10132
    title: 'Open‐Source System Suitability: Mass Spectrometry Query Language Lab (MassQLab)'
  source_content_hash: sha256:32db8ce098798f3d6a28773ec6cf245f5bb32f7477b230389b433fccbd1c0afb
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
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8666666666666667
n_claims: 15
---

# collision-energy-response-curve-fitting

## Summary

Quantifies how an MS/MS product ion (e.g., m/z 85.0281/85.0824 with 10 ppm tolerance in the deuterated acylcarnitine SST mixture) responds to normalized collision energy (NCE 10–60) under CID versus HCD, then fits a logarithmic signal-versus-NCE model per fragmentation mode. Built on MassQL queries executed via MassQLab over mzML data.

## When to use

Use when DDA LC-MS/MS mzML files contain scans of the same precursor at multiple NCE steps and under both CID and HCD, and you need to compare product ion signal (e.g., m/z 85.0824, 10 ppm) versus NCE between fragmentation modes.

## When NOT to use

- Input is already a feature table or XIC peak list with no MS2 scans — MassQL requires mzML spectral data
- The goal is peak intensity and peak shape assessment of the MS1 SST analytes — the article explicitly did not evaluate intensity and peak shape in the MS/MS assessment
- Only a single NCE or a single fragmentation mode is available — the CID-vs-HCD log-fit comparison requires matched scans across NCE 10-60 in both modes

## Inputs

- Vendor .raw LC-MS/MS DDA files of the deuterated acylcarnitine SST mixture (MassIVE MSV000098825)
- mzML files converted with MSConvert default settings
- MassQL query file (JSON, CSV, or XLSX) with named MS2 queries (name + query fields)
- massqlab_config. specifying data_directory, queryfile, and output settings

## Outputs

- ms2_raw_df.csv / ms2_analysis_df.csv with per-scan m/z 85.0824 intensities and metadata
- ms2_plots.pdf, ms2_summary_plots.pdf, and ms2_cluster_plots.pdf (MS2 analysis clustered by energy level)
- product_ion_signal_vs_nce.xlsx tabulating product ion peak areas at NCE 10-60 for CID and HCD
- Signal-versus-NCE plots and fitted logarithmic models per fragmentation mode (matplotlib/reportlab PDF report)

## How to apply

Convert vendor .raw files to mzML with MSConvert using default settings (MassQLab's built-in conversion is experimental). Author an MS2 MassQL query of the form QUERY scaninfo(MS2DATA) WHERE MS2PREC=<precursor>:TOLERANCEPPM=2.5 AND RTMIN/RTMAX=<window> FILTER MS2PROD=85.0824:TOLERANCEPPM=10, place it with a unique name in a JSON/CSV/XLSX query file, and point massqlab_config.'s data_directory and queryfile at the mzML folder and query file. Run MassQLab (Python 3.9, e.g., src\MassQLab_console.py) so each query is applied to each mzML file and the m/z 85.0824 product ion peak area is computed per scan at NCE 10, 20, 30, 40, 50, and 60 under both HCD and CID. Fit a logarithmic model of product ion signal versus NCE separately for CID and HCD, and compare modes at each of the six NCE steps; tabulate areas (e.g., product_ion_signal_vs_nce.xlsx via openpyxl) and render signal-versus-NCE plots with matplotlib. Because DDA MS/MS triggering is stochastic, normalize to total signal when assessing consistency of the comparison.

## Related tools

- **MassQLab** (Python 3.9 Jupyter/console implementation of MassQL that applies the MS2 product-ion queries to a directory of mzML files and tabulates/visualizes results) — https://github.com/JohnsonDylan/MassQLab
- **MassQL** (Query language used to select MS2 scans by precursor (2.5 ppm), RT window, and product ion m/z 85.0824 (10 ppm) for each NCE step) — https://github.com/mwang87/MassQueryLanguage
- **MSConvert** (Converts vendor .raw SST files to mzML (default settings) prior to MassQL querying) — https://proteowizard.sourceforge.io/
- **Python** (Runtime for MassQLab, logarithmic NCE fits, and result tabulation (openpyxl, matplotlib, reportlab))

## Examples

```
massql sample.mzML "QUERY scaninfo(MS2DATA) WHERE MS2PREC=429.3765:TOLERANCEPPM=2.5 AND RTMIN=9.0 AND RTMAX=9.5 FILTER MS2PROD=85.0281:TOLERANCEPPM=10" --output_file results.tsv
```

## Evaluation signals

- The CID m/z 85.0824 signal is lower than the HCD signal at every NCE step tested (10, 20, 30, 40, 50, 60)
- Logarithmic model of product ion signal versus NCE fits both CID and HCD data (signal increases logarithmically with NCE)
- Every scan used passes the MassQL filters: precursor within 2.5 ppm, RT window respected, product ion within 10 ppm of m/z 85.0281/85.0824
- MassQLab output directory contains expected ms2 outputs (ms2_raw_df.csv, ms2_analysis_df.csv, ms2_cluster_plots.pdf) and peaks meet detection criteria to be considered valid
- Results reproduce consistently when cache_setting=true triggers re-querying from the cached Feather file

## Limitations

- The stochastic nature of MS/MS triggering using DDA is problematic for assessing consistency using SST; normalization to total signal is needed
- MassQLab's built-in convert_raw feature is experimental; convert files with ProteoWizard msconvert beforehand
- The article notes that SST MS/MS evaluation was not warranted in the authors' use case (reproducibility in product ions and relative intensity was already high)
- MassQL performance is tested in Python 3.9; other versions may not work

## Evidence

- [readme] Get MS2 scans with a precursor ion matching m/z 429.3765, retention time between 9.0 and 9.5 minutes, and return intensity of the peak with m/z 85.0281: "QUERY scaninfo(MS2DATA) WHERE MS2PREC=429.3765:TOLERANCEPPM=2.5 AND RTMIN=9.0 AND RTMAX=9.5 FILTER MS2PROD=85.0281:TOLERANCEPPM=10"
- [methods] Raw data converted to mzML before MassQL use: "Raw data files (.raw) were converted to mzML format using MSConvert (https://proteowizard.sourceforge.io/) with default settings."
- [discussion] DDA stochasticity is a limitation for consistency assessment: "the stochastic nature of MS/MS triggering using DDA is problematic for assessing consistency using SST, albeit normalization to the total signal helped in our evaluation."
- [intro] MassQLab applies queries to mzML directory and tabulates results: "MassQLab applies a series of queries (written in the language of MassQL) to a directory containing mass spectrometry data in mzML format. Results are tabulated and saved as images and csv files."
- [readme] MS2 outputs include energy-clustered summaries: "ms2_cluster_plots.pdf: Summary of MS2 analysis clustered by energy level if applicable"
