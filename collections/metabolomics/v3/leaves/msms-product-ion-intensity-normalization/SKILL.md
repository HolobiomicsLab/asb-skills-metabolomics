---
name: msms-product-ion-intensity-normalization
description: Use when assessing day-to-day MS/MS product ion consistency from DDA-acquired
  system suitability (SST) data where stochastic precursor triggering makes raw product
  ion intensities incomparable across scans and acquisitions.
when_to_use_negative:
- When only MS1-level metrics (retention time, extracted ion chromatogram peak areas)
  are needed — MS1 queries suffice without product-ion normalization
- When MS/MS scans are triggered deterministically (e.g. targeted PRM/SRM) rather
  than stochastically by DDA, since the normalization rationale targets DDA triggering
  variability
- When the goal is absolute quantification of fragment ion intensities — normalization
  to total scan signal deliberately discards absolute intensity information
edam_operation: http://edamontology.org/operation_3637
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: MassQLab
  role: Python/Jupyter implementation that applies named MassQL MS2 queries to a directory
    of mzML files and tabulates product ion intensities for normalization and export
  repo: https://github.com/JohnsonDylan/MassQLab
- name: MassQL
  role: Query language used to capture DDA MS/MS scans by precursor (MS2PREC, 2.5
    ppm) and extract diagnostic product ion intensities (MS2PROD, 10 ppm)
  repo: https://github.com/mwang87/MassQueryLanguage
- name: MSConvert
  role: Converts vendor .raw SST files to mzML with default settings prior to MassQL
    querying
  repo: https://proteowizard.sourceforge.io/
- name: Python
  role: Language in which MassQLab is implemented (Python 3.9) and used for normalization
    and RSD computation
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1002/rcm.10132
    title: 'Open‐Source System Suitability: Mass Spectrometry Query Language Lab (MassQLab)'
  source_content_hash: sha256:a52f39914b9ed4a436052ee35cacfff0ab307c80b07f88cb99df0405276c7df7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/JohnsonDylan/MassQLab
  tools:
  - MSConvert
  - MassQL
  - MassQLab
  - Python
  edam_operation: http://edamontology.org/operation_3637
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 14
---

# msms-product-ion-intensity-normalization

## Summary

Normalizes DDA MS/MS product ion intensities to the total signal of each product ion scan so that relative intensities of diagnostic fragments (e.g. carnitine fragment m/z 85.0281/85.0824) can be compared across repeated SST acquisitions, revealing consistent trends such as logarithmic increase of m/z 85.0824 under CID and HCD.

## When to use

Use when assessing day-to-day MS/MS product ion consistency from DDA-acquired system suitability (SST) data where stochastic precursor triggering makes raw product ion intensities incomparable across scans and acquisitions.

## When NOT to use

- When only MS1-level metrics (retention time, extracted ion chromatogram peak areas) are needed — MS1 queries suffice without product-ion normalization
- When MS/MS scans are triggered deterministically (e.g. targeted PRM/SRM) rather than stochastically by DDA, since the normalization rationale targets DDA triggering variability
- When the goal is absolute quantification of fragment ion intensities — normalization to total scan signal deliberately discards absolute intensity information

## Inputs

- DDA LC-MS/MS mzML files of a system suitability mixture (e.g. six deuterated acylcarnitine standards, MassIVE MSV000098825)
- MassQL query file (JSON, CSV, or XLSX) with named per-analyte MS2PREC and MS2PROD queries
- massqlab_config. specifying data_directory, queryfile, and output settings

## Outputs

- Tabulated MS2 product ion intensities per scan/query (ms2_raw_df.csv, exportable as .xlsx)
- Normalized product ion relative intensities and RSDs of product ion presence and relative intensity across acquisitions
- MS2 summary plots (ms2_plots.pdf, ms2_summary_plots.pdf, ms2_cluster_plots.pdf)

## How to apply

Acquire SST mixture LC-MS/MS (DDA) data on deuterated acylcarnitine standards and convert raw files to mzML with MSConvert default settings. Write named MassQL MS2 queries per analyte (e.g. QUERY scaninfo(MS2DATA) WHERE MS2PREC=<precursor>:TOLERANCEPPM=2.5) and, for diagnostic fragments, add product-ion filters such as MS2PROD=85.0281:TOLERANCEPPM=10, matching the paper's MS/MS settings (30000 resolution, 1.5 m/z isolation, HCD/CID at sequential 10–60 NCE). Run MassQLab with massqlab_config. pointing at the mzML directory and query file so query results are tabulated for export. Then normalize each product ion intensity to the intensity of its product ion scan (total scan signal), compute relative standard deviations of product ion presence and relative intensity across the ~6-month, two-platform, five-operator SST series, and evaluate trends (e.g. logarithmic increase of m/z 85.0824 for CID and HCD). Judge success by reproducibility of product ions and their relative intensities after normalization, while recognizing that the stochastic nature of DDA MS/MS triggering means full SST MS/MS evaluation is not warranted for this use case.

## Related tools

- **MassQLab** (Python/Jupyter implementation that applies named MassQL MS2 queries to a directory of mzML files and tabulates product ion intensities for normalization and export) — https://github.com/JohnsonDylan/MassQLab
- **MassQL** (Query language used to capture DDA MS/MS scans by precursor (MS2PREC, 2.5 ppm) and extract diagnostic product ion intensities (MS2PROD, 10 ppm)) — https://github.com/mwang87/MassQueryLanguage
- **MSConvert** (Converts vendor .raw SST files to mzML with default settings prior to MassQL querying) — https://proteowizard.sourceforge.io/
- **Python** (Language in which MassQLab is implemented (Python 3.9) and used for normalization and RSD computation)

## Examples

```
massql sst_d0.mzML "QUERY scaninfo(MS2DATA) WHERE MS2PREC=429.3765:TOLERANCEPPM=2.5 FILTER MS2PROD=85.0281:TOLERANCEPPM=10" --output_file results.tsv
```

## Evaluation signals

- After total-signal normalization, product ion relative intensities show consistent trends across days (e.g. m/z 85.0824 increases logarithmically for both CID and HCD)
- Relative standard deviations of product ion presence and relative intensity are computed across the ~6-month, two-platform, five-operator acquisition series and are small enough to indicate reproducibility
- MassQL queries return MS2 scans only when the precursor matches within 2.5 ppm tolerance (MS2PREC) and product ions within 10 ppm (MS2PROD)
- MassQLab writes ms2_raw_df.csv and downstream analysis outputs organized by timestamp in MassQLab_Output
- Qualitative check matches the paper's conclusion: product ions and relative intensities are reproducible, but SST MS/MS evaluation remains unwarranted due to stochastic DDA triggering

## Limitations

- The stochastic nature of MS/MS triggering using DDA is problematic for assessing consistency using SST; normalization to total signal helped but SST MS/MS evaluation is not warranted in the authors' use case
- MassQLab's convert_raw feature is experimental; files should be converted with ProteoWizard MSConvert before use
- MassQLab performance and available functions are still to be refined/expanded in future work
- MS/MS triggering variability means only relative (normalized) intensities, not absolute fragment intensities, are comparable across acquisitions

## Evidence

- [results] Day-to-day MS/MS consistency was assessed by normalizing product ion intensity to the intensity of the product ion scan: [evidence span withheld — anchor status: not_found]
- [discussion] Our data indicate reproducibility in the product ions and their relative intensity, and that SST MS/MS evaluation is not warranted in our use case.: "Our data indicate reproducibility in the product ions and their relative intensity, and that SST MS/MS evaluation is not warranted in our use case."
- [discussion] the stochastic nature of MS/MS triggering using DDA is problematic for assessing consistency using SST, albeit normalization to the total signal helped in our evaluation.: "the stochastic nature of MS/MS triggering using DDA is problematic for assessing consistency using SST, albeit normalization to the total signal helped in our evaluation."
- [methods] Raw data files (.raw) were converted to mzML format using MSConvert (https://proteowizard.sourceforge.io/) with default settings.: "Raw data files (.raw) were converted to mzML format using MSConvert (https://proteowizard.sourceforge.io/) with default settings."
- [readme] MassQLab applies a series of queries (written in the language of MassQL) to a directory containing mass spectrometry data in mzML format. Results are tabulated and saved as images and csv files.: "MassQLab applies a series of queries (written in the language of MassQL) to a directory containing mass spectrometry data in mzML format. Results are tabulated and saved as images and csv files."
