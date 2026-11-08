---
name: lc-ms-raw-data-peak-picking
description: Use when raw LC-MS mzXML files (e.g., DDA data from an exposome or metabolomics
  mixture) need conventional XCMS peak picking as a feature table before JPA's MS2-recognition
  or targeted-list extraction steps.
when_to_use_negative:
- Input is already an aligned feature table — JPA requires per-file CSV feature tables
  for multi-sample analysis, and aligned tables are not acceptable
- Running find.level3features for MR features on full-scan or DIA data is inappropriate;
  but note the PP step itself is the required first stage — do not skip it when only
  MS2-based rescue is wanted
edam_operation: http://edamontology.org/operation_3214
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: JPA
  role: R package providing XCMS.featureTable() to run embedded XCMS peak picking
    and output the PP feature table
  repo: https://github.com/HuanLab/JPA.git
- name: xcms
  role: Embedded peak-picking algorithms called by JPA to extract MS1 PP features
    from raw LC-MS data
- name: devtools
  role: Used to install JPA from GitHub (install_github("HuanLab/JPA"))
- name: BiocManager
  role: Dependency installer for the R/Bioconductor environment used with JPA
- name: RStudio
  role: Recommended environment for installing and running JPA
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.3390/metabo12030212
    title: 'JPA: Joint Metabolic Feature Extraction Increases the Depth of Chemical
      Coverage for LC-MS-Based Metabolomics and Exposomics'
  source_content_hash: sha256:e4c5e85fbf4990432e32d682936004f4aaa933642397e822427524438defb319
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/HuanLab/JPA.git
  tools:
  - BiocManager
  - JPA
  - R
  - RStudio
  - devtools
  - xcms
  edam_operation: http://edamontology.org/operation_3214
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.46153846153846156
n_claims: 13
---

# lc-ms-raw-data-peak-picking

## Summary

Extract MS1 peak-picking (PP) metabolic features from raw LC-MS mzXML data using JPA's embedded XCMS functions, producing a dataframe feature table with mz, rt, rtmin, rtmax, maxo, sample, and level columns that serves as the baseline for downstream MR/targeted feature rescue.

## When to use

Use when raw LC-MS mzXML files (e.g., DDA data from an exposome or metabolomics mixture) need conventional XCMS peak picking as a feature table before JPA's MS2-recognition or targeted-list extraction steps.

## When NOT to use

- Input is already an aligned feature table — JPA requires per-file CSV feature tables for multi-sample analysis, and aligned tables are not acceptable
- Running find.level3features for MR features on full-scan or DIA data is inappropriate; but note the PP step itself is the required first stage — do not skip it when only MS2-based rescue is wanted

## Inputs

- One or multiple mzXML files from LC-MS analyses (any vendor format compatible with MS-Convert), placed in a single dedicated folder
- Optionally, a custom CSV feature table (columns: m/z, rt, rtmin, rtmax, intensity, with RTs in seconds) processed via custom.featureTable()

## Outputs

- A dataframe-formatted feature table with columns mz, rt, rtmin, rtmax, maxo, sample, and a 'level' column labeling features as 'PP'
- An MSnbase object carrying the raw data for downstream JPA steps

## How to apply

Place all mzXML files for the analysis in a single folder containing no other irrelevant mzXML files, and read them via JPA's XCMS.featureTable(), which returns a dataframe feature table plus an MSnbase object; the 'level' column labels each feature as 'PP'. For the exposome standard mixture baseline in this study, peak picking was run with ppm=10, peakwidth=c(5,20), mzdiff=0.01, snthresh=6, prefilter=c(3,100), and noise=100. JPA accepts any vendor format compatible with MS-Convert. For multi-sample analyses, apply this PP extraction per file first, then perform sample alignment afterwards with peak.alignment() (e.g., bw=5, minfrac=0.5, mzwid=0.015). This PP-only table is the baseline against which MR features (find.level3features, level3.threshold=2) and targeted-list features (find.targetfeatures, target.threshold=2) are added, so counts of compounds/features detected with PP alone versus full JPA directly quantify the rescue gain.

## Related tools

- **JPA** (R package providing XCMS.featureTable() to run embedded XCMS peak picking and output the PP feature table) — https://github.com/HuanLab/JPA.git
- **xcms** (Embedded peak-picking algorithms called by JPA to extract MS1 PP features from raw LC-MS data)
- **devtools** (Used to install JPA from GitHub (install_github("HuanLab/JPA")))
- **BiocManager** (Dependency installer for the R/Bioconductor environment used with JPA)
- **RStudio** (Recommended environment for installing and running JPA)

## Examples

```
featureTable <- XCMS.featureTable(dir = dir, ppm=10, peakwidth=c(5,20), mzdiff=0.01, snthresh=6, prefilter=c(3,100), noise=100)
```

## Evaluation signals

- Feature table contains the required columns (mz, rt, rtmin, rtmax, maxo, sample, level) with level='PP' for all extracted features
- XCMS parameters match the study baseline: ppm=10, peakwidth=c(5,20), mzdiff=0.01, snthresh=6, prefilter=c(3,100), noise=100
- For the MTBLS2631 exposome mixture, PP-only compound counts per concentration level are strictly lower than full-JPA counts (JPA detects ~2.3-fold more exposure compounds on average)
- Folder contains only the mzXML files of interest; no irrelevant mzXML files cause extra features or errors
- Feature table feeds cleanly into peak.alignment() with bw=5, minfrac=0.5, mzwid=0.015, minsamp=1, max=100 for multi-sample studies

## Limitations

- Conventional peak picking misses features with low signal intensities, poor chromatographic peak shapes, or those that do not fit the parameter settings — this is exactly why JPA adds MR and targeted-list extraction afterwards
- Feature tables generated by external software must follow the strict column order (m/z, rt, rtmin, rtmax, intensity) with RTs in seconds, and for multi-sample analysis each CSV must be per single mzXML file, not aligned
- Raw data must be convertible to mzXML via MS-Convert; other formats are not directly accepted

## Evidence

- [methods] XMS feature extraction call: "The XCMS.featureTable() function outputs a dataframe formatted featuretable as well as an MSnbase object."
- [methods] PP labeling: [evidence span withheld — anchor status: not_found]
- [readme] Parameter baseline: "featureTable <- XCMS.featureTable(dir = dir, mz.tol = 10, ppm=10, peakwidth=c(5,20), mzdiff = 0.01,
                             snthresh = 6, integrate = 1, prefilter = c(3,100), noise = 100)"
- [readme] Single-folder input requirement: "All mzXML file(s) need
to be placed in the same folder containing no other irrelevant mzXML
files."
- [abstract] Why PP alone is insufficient: "Conventional feature extraction algorithms fail to recognize features with low signal intensities, poor chromatographic peak shapes, or those that do not fit the parameter settings."
