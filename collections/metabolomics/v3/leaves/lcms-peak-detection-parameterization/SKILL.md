---
name: lcms-peak-detection-parameterization
description: Use when raw LC-MS/MS data (converted to mzML) from a Multi-ABLE extraction
  must be converted into a processed, aligned peak table with adduct annotations before
  downstream filtering, normalization, or joint metabolomic–lipidomic analysis.
when_to_use_negative:
- Input is already a processed feature table — this skill is the upstream XCMS pre-processing
  step; downstream steps (LipidFinder, metID, filtering, EigenMS normalization) take
  over after runXCMS output exists
- Proteomic data output from the MultiABLER extraction method — MultiABLER currently
  does not provide functions to analyze proteomic data
- Data requiring kNN imputation, mean normalization, or quantile normalization — these
  manipulation/normalization methods were not implemented in MultiABLER
edam_operation: http://edamontology.org/operation_3214
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: MultiABLER
  role: Provides the runXCMS R wrapper that configures the peak-detection, alignment,
    and grouping steps and emits the feature table
  repo: https://github.com/holab-hku/MultiABLER
- name: R
  role: Execution environment; runXCMS is run from R and MultiABLER is installed as
    an R package via devtools
- name: xcms
  role: R package supplying the centWave peak detection, Obiwarp retention-time alignment,
    and peak-density grouping algorithms invoked by runXCMS
- name: CAMERA
  role: R package whose annotate step adds putative adduct annotations to the grouped
    features
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.isci.2023.106881
    title: Integrative processing of untargeted metabolomic and lipidomic data using
      MultiABLER
  source_content_hash: sha256:07081e8007565f81026fdbf78f993271c17a706f8cf1356900ffa22b2b975f07
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/holab-hku/MultiABLER
  repo_url: https://github.com/holab-hku/MultiABLER
  tools:
  - CAMERA
  - MultiABLER
  - R
  - xcms
  edam_operation: http://edamontology.org/operation_3214
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# lcms-peak-detection-parameterization

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Parameterize the LC-MS/MS peak-processing step of the MultiABLER pipeline by configuring the XCMS runXCMS wrapper — centWave peak detection, Obiwarp retention-time alignment, peak-density grouping, and CAMERA adduct annotation — so raw mzML files become an aligned, adduct-annotated feature table (feature_table.csv).

## When to use

Use when raw LC-MS/MS data (converted to mzML) from a Multi-ABLE extraction must be converted into a processed, aligned peak table with adduct annotations before downstream filtering, normalization, or joint metabolomic–lipidomic analysis.

## When NOT to use

- Input is already a processed feature table — this skill is the upstream XCMS pre-processing step; downstream steps (LipidFinder, metID, filtering, EigenMS normalization) take over after runXCMS output exists
- Proteomic data output from the MultiABLER extraction method — MultiABLER currently does not provide functions to analyze proteomic data
- Data requiring kNN imputation, mean normalization, or quantile normalization — these manipulation/normalization methods were not implemented in MultiABLER

## Inputs

- Raw LC-MS/MS data converted to mzML files (via MSConvert) from Multi-ABLE extraction runs

## Outputs

- Aligned, adduct-annotated feature table saved as feature_table.csv (one row per grouped feature, non-empty adduct column)

## How to apply

Load the mzML files (converted from raw vendor data via MSConvert) into the R/xcms environment and run the MultiABLER runXCMS wrapper. Within the wrapper, detect chromatographic peaks with the centWave algorithm (configuring ppm mass tolerance and the peakwidth range as the key centWave parameters), align retention times across samples using the Obiwarp method, and group chromatographic peaks across samples with the peak-density (groupDensity) method. Then annotate detected features with putative adducts using the CAMERA annotate step and assemble the result into an aligned, adduct-annotated feature table saved as feature_table.csv. The rationale is that this parameterized XCMS front-end is the specific pre-processing stage of MultiABLER that downstream LipidFinder filtering, metID annotation, filtering, and EigenMS normalization depend on, so centWave/Obiwarp/groupDensity settings directly determine feature-table quality.

## Related tools

- **MultiABLER** (Provides the runXCMS R wrapper that configures the peak-detection, alignment, and grouping steps and emits the feature table) — https://github.com/holab-hku/MultiABLER
- **R** (Execution environment; runXCMS is run from R and MultiABLER is installed as an R package via devtools)
- **xcms** (R package supplying the centWave peak detection, Obiwarp retention-time alignment, and peak-density grouping algorithms invoked by runXCMS)
- **CAMERA** (R package whose annotate step adds putative adduct annotations to the grouped features)

## Examples

```
devtools::install_github("holab-hku/MultiABLER", dependencies = TRUE); peaks <- runXCMS(files = list.files(pattern=".mzML$", full.names=TRUE))
```

## Evaluation signals

- The produced feature_table.csv contains one row per grouped feature with a non-empty adduct column (CAMERA annotations present)
- Retention times are aligned across samples (Obiwarp applied) — features appear at consistent retention-time windows across injections
- Feature table structure matches the MultiABLER pre-processing output expected by subsequent LipidFinder filtering and metID annotation steps
- Feature table is suitable as input for differential expression analysis, sPLS, and rCCA downstream analyses
- centWave parameters (ppm tolerance, peakwidth range) are explicitly set and recorded as in the MultiABLER runXCMS wrapper rather than left at unspecified defaults

## Limitations

- MultiABLER currently does not provide functions to analyze proteomic data, which is another important output from the MultiABLER extraction method; proteomic integration is planned for future development
- Other data manipulation and normalization methods, such as kNN imputation, mean normalization, and quantile normalization, were not implemented in MultiABLER
- Raw data must first be converted to mzML via MSConvert before the XCMS step; native vendor formats are not consumed directly by the runXCMS wrapper

## Evidence

- [results] In the MultiABLER workflow, raw LC-MS/MS data (converted to mzML via MSConvert) are first processed with XCMS, then filtered with LipidFinder, annotated with metID, and filtered and normalized, with peaks identified using the runXCMS function.: [evidence span withheld — anchor status: not_found]
- [results] Peaks are identified using the runXCMS function; the wrapper applies centWave peak detection, Obiwarp retention-time alignment, peak-density grouping, and CAMERA adduct annotation before assembling feature_table.csv.: [evidence span withheld — anchor status: not_found]
