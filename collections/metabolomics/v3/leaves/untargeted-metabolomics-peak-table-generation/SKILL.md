---
name: untargeted-metabolomics-peak-table-generation
description: Use when raw LC-MS untargeted metabolomics chromatograms (.CDF files)
  plus a sample metadata table are available and the goal is to produce a features
  x samples peak-area/intensity table with retention-time alignment before imputation
  or correction steps.
when_to_use_negative:
- Input is already a peak/feature intensity table from another processing tool — the
  integration and RT alignment steps of 2. Integration.R operate on raw .CDF chromatograms
- The analysis goal is downstream statistical analysis or biomarker testing — that
  is handled by later OUKS steps (e.g. 9. Statistics.R), not the integration step
- The data are GC-MS or MS/MS-centered workflows — OUKS Integration is scoped to LC-MS
  untargeted metabolomic profiling
edam_operation: http://edamontology.org/operation_3216
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0092
tools:
- name: OUKS
  role: Provides 2. Integration.R, the second of nine scripts performing peaks integration
    and time alignment on raw LC-MS data
  repo: https://github.com/plyush1993/OUKS
- name: R (>= 4.1.2)
  role: Runtime environment in which the integration/alignment script is executed
  repo: https://cloud.r-project.org/
- name: RStudio
  role: IDE used to run OUKS scripts, which are adapted to a tidy format suitable
    for the RStudio headings system
  repo: https://www.rstudio.com/products/rstudio/download/
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/acs.jproteome.1c00392
    title: '<i>Omics Untargeted Key Script</i>: R-Based Software Toolbox for Untargeted
      Metabolomics with Bladder Cancer Biomarkers Discovery Case Study'
  source_content_hash: sha256:c812fd7f45778e80687201b5e6e5da07b32035eb5f8d6bd4368417cc25f0ce96
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/plyush1993/OUKS
  tools:
  - OUKS
  - R
  - R (>= 4.1.2)
  - RStudio
  edam_operation: http://edamontology.org/operation_3216
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# untargeted-metabolomics-peak-table-generation

## Summary

Generates an aligned LC-MS peak intensity table from raw untargeted metabolomics chromatograms by integrating target peaks in extracted-ion chromatograms (EICs) and aligning retention times across runs, as implemented in step 2 (2. Integration.R) of the OUKS nine-step toolbox.

## When to use

Use when raw LC-MS untargeted metabolomics chromatograms (.CDF files) plus a sample metadata table are available and the goal is to produce a features x samples peak-area/intensity table with retention-time alignment before imputation or correction steps.

## When NOT to use

- Input is already a peak/feature intensity table from another processing tool — the integration and RT alignment steps of 2. Integration.R operate on raw .CDF chromatograms
- The analysis goal is downstream statistical analysis or biomarker testing — that is handled by later OUKS steps (e.g. 9. Statistics.R), not the integration step
- The data are GC-MS or MS/MS-centered workflows — OUKS Integration is scoped to LC-MS untargeted metabolomic profiling

## Inputs

- Raw LC-MS untargeted profiling chromatograms in .CDF format (e.g. from Metabolomics Workbench study ST001682)
- metadata.csv sample table (run order, sample classes)
- Output of 1. Randomization.R (sample processing order)

## Outputs

- Aligned peak intensity table (.csv; features as rows, samples as columns, one row per detected feature)
- Intermediate R workspace state (.RData)

## How to apply

Run the OUKS '2. Integration.R' script in RStudio under R (>= 4.1.2), after the randomization step (1. Randomization.R) has fixed the processing order. Process the raw .CDF chromatograms in the order established by the randomization step, guided by the metadata.csv table, integrating target peaks in extracted-ion chromatograms (EICs) for every sample to obtain peak areas/intensities. Then perform retention-time alignment across samples so that molecular features from different runs map to consistent RT windows. Assemble the aligned peak intensity table with features as rows and samples as columns, export it as a .csv file alongside the other datasets in .csv format, and save the intermediate state as .RData. This is the second step of OUKS's comprehensive nine-step LC-MS untargeted metabolomic profiling data processing toolbox, feeding downstream imputation (3. Imputation.R) and drift correction (4. Correction.R).

## Related tools

- **OUKS** (Provides 2. Integration.R, the second of nine scripts performing peaks integration and time alignment on raw LC-MS data) — https://github.com/plyush1993/OUKS
- **R (>= 4.1.2)** (Runtime environment in which the integration/alignment script is executed) — https://cloud.r-project.org/
- **RStudio** (IDE used to run OUKS scripts, which are adapted to a tidy format suitable for the RStudio headings system) — https://www.rstudio.com/products/rstudio/download/

## Examples

```
source("2. Integration.R")  # run in RStudio under R >= 4.1.2, after 1. Randomization.R, on ST001682 .CDF files guided by metadata.csv
```

## Evaluation signals

- Output .csv exists and parses as a tabular file with the expected sample/feature fields
- Peak table contains one row per detected feature, with features as rows and samples as columns
- All samples from metadata.csv are represented as columns, in the order established by the randomization step
- Retention-time alignment yields consistent RT windows: each molecular feature from different runs maps to one aligned feature row
- Intermediate .RData state file is saved alongside the .csv output, matching the OUKS Datasets/Auxiliary files structure

## Limitations

- OUKS is Windows OS (desirable) and requires R >= 4.1.2; some packages with strong version dependency are stored in the 'Required packages (archive)' folder, which may hinder reproducibility
- The project status is inactive — stable but no longer actively developed, with support/maintenance only as time allows
- Integration operates on target peaks in EICs; features missed during integration surface later as missing values handled by the imputation step (3. Imputation.R), not here

## Evidence

- [methods] Integration step definition: "peaks integration and time alignment"
- [intro] Toolbox scope: "providing comprehensive nine step LC-MS untargeted metabolomic profiling data processing toolbox"
- [readme] Script table entry: "`2. Integration.R` | peaks integration and time alignment"
- [readme] Raw data source: "Raw data (.CDF format) are available from [Metabolomics Workbench Repository](https://www.metabolomicsworkbench.org/), study ID: [ST001682](http://doi.org/10.21228/M8ZT4C)"
- [readme] Metadata table: "[`Metadata`](https://github.com/plyush1993/OUKS/blob/main/Datasets%20(csv)/metadata.csv) table is also provided"
