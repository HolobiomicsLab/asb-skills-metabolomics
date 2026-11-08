---
name: across-day-rsd-statistics
description: Use when you have repeated SST mixture acquisitions (e.g., deuterated
  acylcarnitine standards at 1 µg/mL) spanning multiple days, operators, or instruments,
  and need RSD-based reproducibility statistics on RT, peak areas, or normalized MS2
  product ion intensities from MassQL query results.
when_to_use_negative:
- Input files are still in vendor .raw format — convert to mzML first (MassQL requires
  mzML; convert_raw is experimental)
- 'The goal is MS/MS-based SST acceptance criteria: the stochastic nature of DDA MS/MS
  triggering makes SST MS/MS evaluation unwarranted in this use case, even after total-signal
  normalization'
- Intensity and peak shape QC of the SST mixture was explicitly not evaluated in the
  illustrative assessment, so this skill is not a substitute for such metrics
edam_operation: http://edamontology.org/operation_3435
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: MassQLab
  role: Python/Jupyter implementation that applies the named MassQL queries to the
    mzML directory, tabulates results, and produces the RT and MS2 analysis dataframes
    from which RSDs are computed
  repo: https://github.com/JohnsonDylan/MassQLab
- name: MassQL
  role: Query language used to extract MS1 RT windows (MS1MZ=207.1418:TOLERANCEPPM=2.5),
    MS2 precursor scans (TOLERANCEPPM=2.5), and diagnostic product ions (MS2PROD=85.0281:TOLERANCEPPM=10)
  repo: https://github.com/mwang87/MassQueryLanguage
- name: MSConvert
  role: Converts vendor .raw SST acquisition files to mzML (default settings) as a
    prerequisite for MassQL querying
  repo: https://proteowizard.sourceforge.io/
- name: Python
  role: Language in which MassQLab (Python 3.9) is implemented and in which the RSD/normalization
    computations can be performed
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1002/rcm.10132
    title: 'Open‐Source System Suitability: Mass Spectrometry Query Language Lab (MassQLab)'
  source_content_hash: sha256:cffcfa8d6e8fefe6cda1e59d0aa81ef324fb37740ae20514396db9d92aa4de4a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/JohnsonDylan/MassQLab
  tools:
  - MSConvert
  - MassQL
  - MassQLab
  - Python
  edam_operation: http://edamontology.org/operation_3435
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 18
---

# across-day-rsd-statistics

## Summary

Compute day-to-day relative standard deviation (RSD) statistics for system suitability testing (SST) metrics — retention time and MS/MS product ion intensities — extracted from LC-HRMS SST mixture data by MassQLab/MassQL queries, following the deuterated acylcarnitine SST workflow (RT RSD 0.5%–2.3% over ~6 months, two platforms, five operators).

## When to use

Use when you have repeated SST mixture acquisitions (e.g., deuterated acylcarnitine standards at 1 µg/mL) spanning multiple days, operators, or instruments, and need RSD-based reproducibility statistics on RT, peak areas, or normalized MS2 product ion intensities from MassQL query results.

## When NOT to use

- Input files are still in vendor .raw format — convert to mzML first (MassQL requires mzML; convert_raw is experimental)
- The goal is MS/MS-based SST acceptance criteria: the stochastic nature of DDA MS/MS triggering makes SST MS/MS evaluation unwarranted in this use case, even after total-signal normalization
- Intensity and peak shape QC of the SST mixture was explicitly not evaluated in the illustrative assessment, so this skill is not a substitute for such metrics

## Inputs

- Directory of mzML files from repeated SST mixture acquisitions (MassIVE MSV000098825; raw files converted with MSConvert default settings)
- massqlab_config. specifying data_directory and queryfile paths
- Query file (JSON, CSV, or XLSX) with named MassQL MS1/MS2 queries per deuterated acylcarnitine SST analyte

## Outputs

- ms1_RT_analysis_df.csv / ms1_analysis_df.csv with retention time RSD statistics per SST analyte
- ms2_raw_df.csv and ms2_analysis_df.csv with scan and product ion intensities (exported as .xlsx per the source workflow)
- Summary plots (ms1_summary_areas.pdf, ms2_cluster_plots.pdf) used to judge reproducibility across days, platforms, and operators

## How to apply

Download the SST LC-MS/MS (DDA) mzML files (MassIVE MSV000098825, raw converted with MSConvert default settings), then run named MassQL queries per SST analyte (e.g., QUERY scaninfo(MS2DATA) WHERE MS2PREC=429.3765:TOLERANCEPPM=2.5, with MS2PROD=85.0281:TOLERANCEPPM=10 product-ion filters) via MassQLab configured through massqlab_config.. For MS1/RT metrics, use MassQLab's Gaussian peak fitting and RT analysis outputs; for MS/MS consistency, add MS2 product-ion filters capturing the diagnostic carnitine fragment under the paper's settings (30000 resolution, 1.5 m/z isolation, HCD/CID at sequential 10–60 NCE). Normalize each MS2 product ion intensity to the total signal of its scan to correct for DDA intensity variation, then compute RSDs of product ion presence and relative intensity across the acquisition series. Judge success by reproducing the paper's ranges: RT RSD of 0.5%–2.3% for all SST analytes and reproducible product ions/relative intensities after total-signal normalization, while concluding SST MS/MS evaluation is not warranted due to stochastic DDA triggering.

## Related tools

- **MassQLab** (Python/Jupyter implementation that applies the named MassQL queries to the mzML directory, tabulates results, and produces the RT and MS2 analysis dataframes from which RSDs are computed) — https://github.com/JohnsonDylan/MassQLab
- **MassQL** (Query language used to extract MS1 RT windows (MS1MZ=207.1418:TOLERANCEPPM=2.5), MS2 precursor scans (TOLERANCEPPM=2.5), and diagnostic product ions (MS2PROD=85.0281:TOLERANCEPPM=10)) — https://github.com/mwang87/MassQueryLanguage
- **MSConvert** (Converts vendor .raw SST acquisition files to mzML (default settings) as a prerequisite for MassQL querying) — https://proteowizard.sourceforge.io/
- **Python** (Language in which MassQLab (Python 3.9) is implemented and in which the RSD/normalization computations can be performed)

## Examples

```
py src\MassQLab_console.py
```

## Evaluation signals

- RT RSD for all SST analytes falls in the 0.5%–2.3% range reported over ~6 months, two instrument platforms, and five operators
- After normalizing product ion intensity to the total signal of its scan, product ions and their relative intensities show consistent trends across days (e.g., m/z 85.0824 increasing logarithmically for both CID and HCD)
- ms1_raw_df.csv and ms2_raw_df.csv contain results for every query × file combination in the query file and data directory (MassQLab processes all files)
- The qualitative acceptance check holds: product ion presence is reproducible, but MS/MS-based SST is judged unwarranted due to stochastic DDA triggering
- Output files (csv, xlsx, PDFs) appear in MassQLab_Output organized by timestamp in the configured output_directory

## Limitations

- The stochastic nature of MS/MS triggering using DDA is problematic for assessing SST consistency; only normalization to total signal made evaluation feasible
- SST MS/MS evaluation is not warranted in the authors' use case (stated limitation), so MS/MS RSDs support qualitative reproducibility checks only
- Intensity and peak shape were not evaluated in the illustrative SST assessment
- MassQLab converts raw files experimentally (convert_raw is flagged as experimental); use MSConvert directly; MassQLab's performance and available functions are still to be refined/expanded

## Evidence

- [results] Statistical assessment of retention time, Table S2, yielded a relative standard deviation (RSD) in all SST analytes ranging from 0.5% to 2.3%: "Statistical assessment of retention time, Table S2, yielded a relative standard deviation (RSD) in all SST analytes ranging from 0.5% to 2.3%"
- [discussion] DDA stochastic triggering limitation: "the stochastic nature of MS/MS triggering using DDA is problematic for assessing consistency using SST, albeit normalization to the total signal helped in our evaluation."
- [discussion] MS/MS unwarranted conclusion: "Our data indicate reproducibility in the product ions and their relative intensity, and that SST MS/MS evaluation is not warranted in our use case."
- [methods] mzML prerequisite: "Raw data files (.raw) were converted to mzML format using MSConvert (https://proteowizard.sourceforge.io/) with default settings."
- [readme] MassQLab tabulation behavior: "MassQLab applies a series of queries (written in the language of MassQL) to a directory containing mass spectrometry data in mzML format. Results are tabulated and saved as images and csv files."
