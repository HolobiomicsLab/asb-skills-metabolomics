---
name: msms-product-ion-quantification
description: Use when LC-MS/MS mzML data (e.g., deuterated acylcarnitine SST mixture
  from MassIVE MSV000098825) must be queried for specific product ion intensities
  or neutral-loss signals across NCE steps and activation modes (CID vs HCD).
when_to_use_negative:
- When the goal is SST evaluation of MS1 metrics only (retention time, extracted ion
  chromatograms) rather than MS/MS product ion quantification — the authors note intensity
  and peak shape were not evaluated and SST MS/MS evaluation was not warranted in
  their use case
- When data cannot be provided as mzML (or converted via MSConvert), since MassQL
  requires the open-source mzML format
- When relying on DDA MS/MS scan counts for consistency assessment without normalization
  — the stochastic nature of DDA triggering is a stated limitation
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: MassQLab
  role: Python 3.9 Jupyter/console implementation that applies MassQL queries to a
    directory of mzML files and tabulates/visualizes product ion results as csv and
    pdf outputs
  repo: https://github.com/JohnsonDylan/MassQLab
- name: MassQL
  role: Domain-specific query language used to express MS2 precursor/product/neutral-loss
    filters (e.g., MS2PREC, MS2PROD, MS2NL with TOLERANCEPPM) executed via the massql
    Python API
  repo: https://github.com/mwang87/MassQueryLanguage
- name: MSConvert
  role: ProteoWizard tool used to convert vendor .raw files to mzML with default settings
    before MassQL querying
  repo: https://proteowizard.sourceforge.io/
provenance:
  source_task_ids:
  - task_003
  - task_004
  source_papers:
  - doi: 10.1002/rcm.10132
    title: 'Open‐Source System Suitability: Mass Spectrometry Query Language Lab (MassQLab)'
  source_content_hash: sha256:cce55f763a2200e8f837d14118924071d7ec0c66b8826d12f03ecb395b7455ea
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
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# msms-product-ion-quantification

## Summary

Quantify MS/MS product ion signals (e.g., m/z 85.0824 or the 85.0281 m/z product of deuterated acylcarnitine SST analytes) across CID and HCD fragmentation modes and normalized collision energies using MassQL queries executed through MassQLab.

## When to use

Use when LC-MS/MS mzML data (e.g., deuterated acylcarnitine SST mixture from MassIVE MSV000098825) must be queried for specific product ion intensities or neutral-loss signals across NCE steps and activation modes (CID vs HCD).

## When NOT to use

- When the goal is SST evaluation of MS1 metrics only (retention time, extracted ion chromatograms) rather than MS/MS product ion quantification — the authors note intensity and peak shape were not evaluated and SST MS/MS evaluation was not warranted in their use case
- When data cannot be provided as mzML (or converted via MSConvert), since MassQL requires the open-source mzML format
- When relying on DDA MS/MS scan counts for consistency assessment without normalization — the stochastic nature of DDA triggering is a stated limitation

## Inputs

- Vendor .raw LC-MS/MS files (converted to .mzML with MSConvert default settings)
- MassQL query file (MassQL_Queries. / .csv / .xlsx) with 'name' and 'query' fields per query
- massqlab_config. specifying data_directory, queryfile, and output options

## Outputs

- ms2_raw_df.csv (raw query results + metadata) and ms2_analysis_df.csv (peak picking analysis) from MassQLab
- ms2_plots.pdf / ms2_summary_plots.pdf / ms2_cluster_plots.pdf visualizations of scan intensities clustered by energy level
- Product-ion-versus-NCE comparison tables/plots (e.g., product_ion_signal_vs_nce.xlsx, tma_neutral_loss_comparison.csv) and fitted log(signal) vs NCE models per activation mode

## How to apply

Convert vendor .raw files to mzML with MSConvert using default settings, then configure massqlab_config. to point at the mzML directory and a query file (JSON/CSV/XLSX) where each query has a unique 'name' and a MassQL string such as 'QUERY scaninfo(MS2DATA) WHERE MS2PREC=<precursor>:TOLERANCEPPM=2.5 AND RTMIN/RTMAX=<window> FILTER MS2PROD=85.0281:TOLERANCEPPM=10'. Run MassQLab (Python 3.9, Jupyter or console entry point) so each query is applied to every mzML file, tabulating product ion peak areas stratified by NCE (10–60) and activation mode; instrument parameters to record include 30000 resolution MS/MS, 1.5 m/z isolation, q = 0.25, 10 ms activation. Compare CID versus HCD signal at each NCE step and, if reproducing the reported behavior, expect CID product ion signal lower than HCD at all NCE values with logarithmic increase in signal versus NCE, and for trimethylamine neutral-loss queries, strong CID signal above NCE 30 with near-undetectable HCD signal.

## Related tools

- **MassQLab** (Python 3.9 Jupyter/console implementation that applies MassQL queries to a directory of mzML files and tabulates/visualizes product ion results as csv and pdf outputs) — https://github.com/JohnsonDylan/MassQLab
- **MassQL** (Domain-specific query language used to express MS2 precursor/product/neutral-loss filters (e.g., MS2PREC, MS2PROD, MS2NL with TOLERANCEPPM) executed via the massql Python API) — https://github.com/mwang87/MassQueryLanguage
- **MSConvert** (ProteoWizard tool used to convert vendor .raw files to mzML with default settings before MassQL querying) — https://proteowizard.sourceforge.io/

## Examples

```
results_df = msql_engine.process_query("QUERY scaninfo(MS2DATA) WHERE MS2PREC=429.3765:TOLERANCEPPM=2.5 FILTER MS2PROD=85.0281:TOLERANCEPPM=10", "sst.mzML")
```

## Evaluation signals

- CID product ion signal at m/z 85.0824 is lower than HCD at every NCE step (10, 20, 30, 40, 50, 60)
- Product ion signal increases logarithmically with NCE for both CID and HCD (log-model fit per mode)
- Trimethylamine neutral-loss queries show strong CID peak area at NCE > 30 while corresponding HCD peak areas are near-undetectable (fail detection criteria)
- Product ion matches fall within the stated tolerances (e.g., MS2PROD=85.0281:TOLERANCEPPM=10, MS2PREC tolerance 2.5 ppm)
- MassQLab produces the expected output set (ms2_raw_df.csv, ms2_analysis_df.csv, ms2_summary_plots.pdf) in MassQLab_Output organized by timestamp

## Limitations

- Stochastic nature of DDA MS/MS triggering is problematic for assessing consistency using SST, albeit normalization to total signal helped in the original evaluation
- The authors note that intensity and peak shape were not evaluated in the illustrative SST assessment
- MassQLab's convert_raw option is experimental; files should be converted with ProteoWizard msconvert beforehand

## Evidence

- [readme] MassQLab applies a series of queries (written in the language of MassQL) to a directory containing mass spectrometry data in mzML format. Results are tabulated and saved as images and csv files.: "MassQLab applies a series of queries (written in the language of MassQL) to a directory containing mass spectrometry data in mzML format. Results are tabulated and saved as images and csv files."
- [methods] Raw data files (.raw) were converted to mzML format using MSConvert (https://proteowizard.sourceforge.io/) with default settings.: "Raw data files (.raw) were converted to mzML format using MSConvert (https://proteowizard.sourceforge.io/) with default settings."
- [methods] QUERY scaninfo(MS2DATA) WHERE MS2PREC=429.3765:TOLERANCEPPM=2.5 AND RTMIN=9.0 AND RTMAX=9.5 FILTER MS2PROD=85.0281:TOLERANCEPPM=10: "QUERY scaninfo(MS2DATA) WHERE MS2PREC=429.3765:TOLERANCEPPM=2.5 AND RTMIN=9.0 AND RTMAX=9.5 FILTER MS2PROD=85.0281:TOLERANCEPPM=10"
- [discussion] the stochastic nature of MS/MS triggering using DDA is problematic for assessing consistency using SST, albeit normalization to the total signal helped in our evaluation.: "the stochastic nature of MS/MS triggering using DDA is problematic for assessing consistency using SST, albeit normalization to the total signal helped in our evaluation."
- [readme] results_df = msql_engine.process_query(input_query, input_filename): [evidence span withheld — anchor status: not_found]
