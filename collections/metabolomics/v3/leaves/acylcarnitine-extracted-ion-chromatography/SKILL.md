---
name: acylcarnitine-extracted-ion-chromatography
description: Use when mzML data from an SST mixture of deuterated acylcarnitine standards
  must be queried by exact mass (ppm tolerance) and RT window to recover per-file
  retention times for RSD-based system suitability assessment.
when_to_use_negative:
- The goal is MS/MS product-ion or neutral-loss evaluation of SST scans (e.g., MS2PREC=429.3765:TOLERANCEPPM=2.5
  queries) rather than MS1 EIC retention-time statistics
- The task requires evaluating peak intensity or peak shape — the authors explicitly
  note 'the intensity and peak shape were not evaluated'
- Data are in vendor-specific .raw format that has not been converted to mzML; convert
  with MSConvert first
edam_operation: http://edamontology.org/operation_3633
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: MassQLab
  role: Python/Jupyter console implementation that applies the named MassQL MS1 queries
    to a directory of mzML files and tabulates peak metrics and RT statistics
  repo: https://github.com/JohnsonDylan/MassQLab
- name: MassQL
  role: Query language used to express the MS1 extracted-ion filters (MS1MZ with TOLERANCEPPM
    and RTMIN/RTMAX)
  repo: https://github.com/mwang87/MassQueryLanguage
- name: MSConvert
  role: Converts vendor .raw files to mzML before querying
  repo: https://proteowizard.sourceforge.io/
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1002/rcm.10132
    title: 'Open‐Source System Suitability: Mass Spectrometry Query Language Lab (MassQLab)'
  source_content_hash: sha256:452fce86937a7315e15e25b72168e32bfbb116a673bc9cf27e70426fbf76a389
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/JohnsonDylan/MassQLab
  tools:
  - MSConvert
  - MassQL
  - MassQLab
  - Python
  edam_operation: http://edamontology.org/operation_3633
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# acylcarnitine-extracted-ion-chromatography

## Summary

Extracts deuterated acylcarnitine SST analytes from UHPLC-HRMS mzML files using MassQL MS1 extracted-ion queries (e.g., MS1MZ=207.1418:TOLERANCEPPM=2.5 with RT windows), then fits chromatographic peaks to compute retention-time RSDs for system suitability.

## When to use

Use when mzML data from an SST mixture of deuterated acylcarnitine standards must be queried by exact mass (ppm tolerance) and RT window to recover per-file retention times for RSD-based system suitability assessment.

## When NOT to use

- The goal is MS/MS product-ion or neutral-loss evaluation of SST scans (e.g., MS2PREC=429.3765:TOLERANCEPPM=2.5 queries) rather than MS1 EIC retention-time statistics
- The task requires evaluating peak intensity or peak shape — the authors explicitly note 'the intensity and peak shape were not evaluated'
- Data are in vendor-specific .raw format that has not been converted to mzML; convert with MSConvert first

## Inputs

- mzML files of SST mixture acquisitions (LC-MS, positive ionization mode, e.g., from MassIVE MSV000098825)
- Query file (JSON, CSV, or XLSX) with named MassQL MS1 extracted-ion queries per deuterated acylcarnitine analyte
- massqlab_config.json with data_directory and queryfile paths

## Outputs

- ms1_raw_df.csv with raw MassQL query results and metadata
- ms1_analysis_df.csv with peak fitting analysis
- ms1_RT_analysis_df.csv with RT statistics vs. query RT
- ms1_traces.pdf / ms1_summary_traces.pdf Gaussian-fit chromatogram images
- Per-analyte retention-time RSD values (expected range 0.5%–2.3%)

## How to apply

Download the SST mixture mzML files (MassIVE MSV000098825; raw .raw files converted with MSConvert default settings). Define one named MassQL query per deuterated acylcarnitine in the queryfile, using an MS1 extracted-ion form such as QUERY scaninfo(MS1DATA) FILTER MS1MZ=207.1418:TOLERANCEPPM=2.5 AND RTMIN=1.0 AND RTMAX=1.2, adjusting the m/z and RTMIN/RTMAX per analyte. Configure massqlab_config.json with data_directory and queryfile, then run MassQLab_console.py, which applies each query to every mzML file, performs peak analysis, and writes ms1_raw_df.csv, ms1_analysis_df.csv, and ms1_RT_analysis_df.csv. Extract each analyte's detected retention time per acquisition file and compute the per-analyte RSD across acquisitions spanning ~6 months, two platforms (Orbitrap Fusion and Fusion Lumos), and five operators. Rationale: exact-mass filtering at 2.5 ppm plus an RT window isolates each deuterated standard's EIC so that RT variability — deemed acceptable at 0.5%–2.3% by the authors despite uncontrolled variables (personnel, columns, LC solvents) — can be quantified without vendor-specific software.

## Related tools

- **MassQLab** (Python/Jupyter console implementation that applies the named MassQL MS1 queries to a directory of mzML files and tabulates peak metrics and RT statistics) — https://github.com/JohnsonDylan/MassQLab
- **MassQL** (Query language used to express the MS1 extracted-ion filters (MS1MZ with TOLERANCEPPM and RTMIN/RTMAX)) — https://github.com/mwang87/MassQueryLanguage
- **MSConvert** (Converts vendor .raw files to mzML before querying) — https://proteowizard.sourceforge.io/

## Examples

```
QUERY scaninfo(MS1DATA) FILTER MS1MZ=207.1418:TOLERANCEPPM=2.5 AND RTMIN=1.0 AND RTMAX=1.2
```

## Evaluation signals

- Each per-analyte retention-time RSD falls within the reported 0.5%–2.3% range (Table S2): minimum ≈0.5%, maximum ≈2.3%
- ms1_RT_analysis_df.csv contains an RT statistics row for every named query in the queryfile and every mzML file in data_directory
- Detected RT for each analyte lies inside the RTMIN–RTMAX window specified in its MassQL query
- Gaussian fits in ms1_traces.pdf converge for each query/file pair (peak must meet detection criteria to be considered valid)
- Output directory contains the expected MassQLab_Output artifacts (ms1_raw_df.csv, ms1_analysis_df.csv, ms1_traces.pdf) organized by timestamp

## Limitations

- Intensity and peak shape were not evaluated in the illustrative SST assessment
- The authors only validated a mixture of six deuterated acylcarnitine standards on Vanquish Horizon-Orbitrap Fusion and Fusion Lumos UHPLC-HRMS; other analyte classes or instruments are untested here
- Only 2% of surveyed untargeted metabolomics papers used SST samples, so comparable benchmark RSD ranges may not exist for other contexts

## Evidence

- [methods] MS1 extracted-ion query with 2.5 ppm tolerance and RT window: "QUERY scaninfo(MS1DATA) FILTER MS1MZ=207.1418:TOLERANCEPPM=2.5 AND RTMIN=1.0 AND RTMAX=1.2"
- [results] Retention time RSD range across SST analytes: "Statistical assessment of retention time, Table S2, yielded a relative standard deviation (RSD) in all SST analytes ranging from 0.5% to 2.3%"
- [methods] SST mixture preparation concentration: "The SST mixture was prepared to a final concentration of 1 μg/mL via the addition of each deuterated standard into 100 mL volumetric glassware"
- [readme] MassQLab outputs RT statistics to CSV: "ms1_RT_analysis_df.csv: RT stats vs. query RT"
- [intro] mzML prerequisite for MassQL: "One prerequisite is that vendor‐specific file formats must be converted into an open‐source MS file format (i.e., .mzML)."
- [results] Intensity/peak shape not evaluated: "Note, the intensity and peak shape were not evaluated."
