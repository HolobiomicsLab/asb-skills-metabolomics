---
name: dda-acquisition-variability-analysis
description: Use when DDA LC-MS/MS system suitability data (e.g., SST acylcarnitine
  mixtures acquired across months, platforms, and operators) needs assessment of MS/MS
  product ion presence and relative intensity consistency despite stochastic DDA scan
  triggering.
when_to_use_negative:
- The goal is MS1-level SST metrics (retention time, peak area/shape, extracted ion
  chromatograms) rather than MS/MS product ion consistency — the paper reports RT
  RSDs of 0.5%-2.3% and explicitly notes intensity and peak shape were not evaluated
  in the MS/MS context.
- Data were acquired in targeted MS/MS (PRM/SRM) mode where scan triggering is deterministic
  — the 'not warranted' conclusion stems from the stochastic nature of DDA MS/MS triggering,
  not from MS/MS-based SST being invalid in general.
- Input files are still in vendor-specific .raw format and MSConvert is unavailable
  — MassQLab requires mzML (its built-in convert_raw option is experimental and ProteoWizard
  conversion is recommended instead).
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0593
tools:
- name: MassQLab
  role: Python/Jupyter package that applies a series of named MassQL MS2 queries to
    a directory of mzML files and tabulates/saves results (csv, images) for the SST
    variability assessment
  repo: https://github.com/JohnsonDylan/MassQLab
- name: MassQL
  role: Query language/engine whose MS2DATA queries (MS2PREC, MS2PROD with TOLERANCEPPM,
    RTMIN/RTMAX) capture precursor-specific DDA scans and diagnostic product ion intensities
  repo: https://github.com/mwang87/MassQueryLanguage
- name: MSConvert
  role: Converts vendor .raw SST files to open-source .mzML format, a prerequisite
    before MassQL queries can be applied
  repo: https://proteowizard.sourceforge.io/
- name: Python
  role: Language in which MassQLab (Python 3.9) and the MassQL engine are implemented;
    used for normalization and RSD computation downstream of query results
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1002/rcm.10132
    title: 'Open‐Source System Suitability: Mass Spectrometry Query Language Lab (MassQLab)'
  source_content_hash: sha256:af9b779810f8c55755adde6b4ed10db04f5fd36c4762740ba8f04a2ec37dee23
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/JohnsonDylan/MassQLab
  tools:
  - MSConvert
  - MassQL
  - MassQLab
  - Python
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0593
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# dda-acquisition-variability-analysis

## Summary

Assess day-to-day MS/MS product ion consistency in DDA-acquired system suitability (SST) LC-MS/MS data by capturing precursor-specific MS2 scans with MassQL and normalizing product ion intensities to total scan signal. It reproduces the MassQLab paper's conclusion that, while product ions and relative intensities are reproducible, the stochastic nature of DDA triggering makes SST MS/MS evaluation unwarranted.

## When to use

Use when DDA LC-MS/MS system suitability data (e.g., SST acylcarnitine mixtures acquired across months, platforms, and operators) needs assessment of MS/MS product ion presence and relative intensity consistency despite stochastic DDA scan triggering.

## When NOT to use

- The goal is MS1-level SST metrics (retention time, peak area/shape, extracted ion chromatograms) rather than MS/MS product ion consistency — the paper reports RT RSDs of 0.5%-2.3% and explicitly notes intensity and peak shape were not evaluated in the MS/MS context.
- Data were acquired in targeted MS/MS (PRM/SRM) mode where scan triggering is deterministic — the 'not warranted' conclusion stems from the stochastic nature of DDA MS/MS triggering, not from MS/MS-based SST being invalid in general.
- Input files are still in vendor-specific .raw format and MSConvert is unavailable — MassQLab requires mzML (its built-in convert_raw option is experimental and ProteoWizard conversion is recommended instead).

## Inputs

- SST mixture LC-MS/MS (DDA) .raw files, converted to .mzML via MSConvert with default settings (MassIVE accession MSV000098825)
- massqlab_config. specifying data_directory, queryfile, and output options
- Named MassQL query file (JSON, CSV, or XLSX) with 'name' and 'query' fields per analyte, containing MS2DATA queries (MS2PREC, optional RTMIN/RTMAX, MS2PROD filters)

## Outputs

- ms2_raw_df.csv (raw query results + metadata) and ms2_analysis_df.csv (peak picking analysis)
- ms2_plots.pdf, ms2_summary_plots.pdf, ms2_cluster_plots.pdf visualizations of scan intensities and MS2 analysis
- Normalized product ion intensities (per total-scan signal) and relative standard deviations of product ion presence and relative intensity across the SST acquisition series, exportable as .xlsx

## How to apply

Convert SST .raw files to mzML with MSConvert default settings, then write a query file of named MassQL MS2 queries per deuterated acylcarnitine precursor (e.g., MS2PREC=429.3765:TOLERANCEPPM=2.5, optionally with RTMIN/RTMAX windows) and add MS2 product-ion filters such as MS2PROD=85.0281:TOLERANCEPPM=10 for the diagnostic carnitine fragment, matching the paper's MS/MS settings (30000 resolution, 1.5 m/z isolation, HCD and CID at sequential 10-60 normalized collision energy). Configure massqlab_config. with the mzML data_directory and queryfile, then run MassQLab (console, GUI, or Jupyter) so each query is applied to every mzML file and results are tabulated. Normalize each product ion intensity to the intensity of its product ion scan (total-signal normalization) to make intensities comparable across repeated acquisitions, then compute relative standard deviations of product ion presence and relative intensity across the ~6-month, two-platform, five-operator series. Validate against the paper's acceptance check: product ions and relative intensities reproduce across days (the m/z 85.0824 signal increased logarithmically for both CID and HCD after normalization), but the stochastic DDA triggering means SST MS/MS evaluation is not warranted for this use case.

## Related tools

- **MassQLab** (Python/Jupyter package that applies a series of named MassQL MS2 queries to a directory of mzML files and tabulates/saves results (csv, images) for the SST variability assessment) — https://github.com/JohnsonDylan/MassQLab
- **MassQL** (Query language/engine whose MS2DATA queries (MS2PREC, MS2PROD with TOLERANCEPPM, RTMIN/RTMAX) capture precursor-specific DDA scans and diagnostic product ion intensities) — https://github.com/mwang87/MassQueryLanguage
- **MSConvert** (Converts vendor .raw SST files to open-source .mzML format, a prerequisite before MassQL queries can be applied) — https://proteowizard.sourceforge.io/
- **Python** (Language in which MassQLab (Python 3.9) and the MassQL engine are implemented; used for normalization and RSD computation downstream of query results)

## Examples

```
massql sample.mzML "QUERY scaninfo(MS2DATA) WHERE MS2PREC=429.3765:TOLERANCEPPM=2.5 FILTER MS2PROD=85.0281:TOLERANCEPPM=10" --output_file results.tsv
```

## Evaluation signals

- Retention time RSD across all SST analytes falls in the 0.5%-2.3% range over the ~6-month, two-platform, five-operator acquisition series, confirming comparable acquisitions.
- After normalizing product ion intensity to the intensity of its product ion scan, consistent trends appear across days — e.g., the m/z 85.0824 signal increases logarithmically for both CID and HCD.
- ms2_raw_df.csv contains rows for each named MS2 query applied to each mzML file, with query results plus metadata (non-empty result set per valid precursor query).
- The MS2PROD=85.0281 filter (TOLERANCEPPM=10) returns the diagnostic carnitine fragment intensity consistent with the paper's MS/MS settings (30000 resolution, 1.5 m/z isolation, 10-60 NCE).
- Final validation matches the paper's qualitative conclusion: product ions and their relative intensities are reproducible, yet stochastic DDA triggering is documented as the reason SST MS/MS evaluation is deemed unwarranted.

## Limitations

- The stochastic nature of MS/MS triggering using DDA is problematic for assessing consistency using SST; total-signal normalization helped in the paper's evaluation but does not eliminate this limitation, and SST MS/MS evaluation is not warranted in the authors' use case.
- MassQLab currently processes all files in the configured data_directory, and its built-in raw-to-mzML conversion (convert_raw) is experimental — ProteoWizard MSConvert conversion beforehand is recommended.
- MassQLab performance and available functions are still to be refined/expanded; no MassQLab software version number is provided in the paper's discussion.

## Evidence

- [discussion] the stochastic nature of MS/MS triggering using DDA is problematic for assessing consistency using SST, albeit normalization to the total signal helped in our evaluation.: "the stochastic nature of MS/MS triggering using DDA is problematic for assessing consistency using SST, albeit normalization to the total signal helped in our evaluation."
- [discussion] Our data indicate reproducibility in the product ions and their relative intensity, and that SST MS/MS evaluation is not warranted in our use case.: "Our data indicate reproducibility in the product ions and their relative intensity, and that SST MS/MS evaluation is not warranted in our use case."
- [methods] QUERY scaninfo(MS2DATA) WHERE MS2PREC=429.3765:TOLERANCEPPM=2.5 AND RTMIN=9.0 AND RTMAX=9.5: "QUERY scaninfo(MS2DATA) WHERE MS2PREC=429.3765:TOLERANCEPPM=2.5 AND RTMIN=9.0 AND RTMAX=9.5"
- [methods] Raw data files (.raw) were converted to mzML format using MSConvert (https://proteowizard.sourceforge.io/) with default settings.: "Raw data files (.raw) were converted to mzML format using MSConvert (https://proteowizard.sourceforge.io/) with default settings."
- [readme] MassQLab applies a series of queries (written in the language of MassQL) to a directory containing mass spectrometry data in mzML format.: "MassQLab applies a series of queries (written in the language of MassQL) to a directory containing mass spectrometry data in mzML format."
