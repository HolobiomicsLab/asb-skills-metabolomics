---
name: lc-ms-qc-rsd-calculation
description: Use when a targeted LC-MS peak table (tab-delimited with aliquot, compound,
  area, type, injection_time, batch columns) contains pooled QC samples (SQC/LQC)
  and you need a per-compound QC-reproducibility metric for compound filtering.
when_to_use_negative:
- When a compound is not detected in LQC or SQC samples it is assumed not present
  and will not be reported by mzQuality, so RSDQC cannot be computed for it
- Data that have not been peak-integrated to a compound-by-sample area table (mzQuality
  processes peak-integrated data independently of vendor software and does not process
  raw peaks like OpenMS or MZmine)
- When sample-level outliers, not compound QC reproducibility, are the target — the
  Rosner Test on QC samples is the corresponding sample-based check
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: mzQuality
  role: Computes compound/IS ratios, batch-corrects against pooled SQC samples, and
    calculates the RSD of QC samples (RSDQC) per compound for quality classification
  repo: https://github.com/hankemeierlab/mzQuality
- name: R
  role: Runtime environment (R ≥ 4.0) in which mzQuality is distributed and executed
- name: mzQualityDashboard
  role: Interactive Shiny front-end recommended for running the same RSDQC-based analysis
    without writing R code
  repo: https://github.com/hankemeierlab/mzQualityDashboard
provenance:
  source_task_ids:
  - task_005
  - task_004
  source_papers:
  - doi: 10.1021/jasms.5c00073
    title: 'mzQuality: An Open-Source Software Tool for Quality Monitoring and Reporting
      of Targeted Mass Spectrometry Measurements'
  source_content_hash: sha256:80dd377d01660c6850f1aa3747e01e78f0386baed0c3919f5ff4e09136ed3ddb
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/hankemeierlab/mzQuality
  tools:
  - R
  - mzQuality
  - mzQualityDashboard
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 14
---

# lc-ms-qc-rsd-calculation

## Summary

Computes the relative standard deviation of pooled QC sample peak-area ratios (RSDQC) per compound in mzQuality, after batch correction and outlier exclusion, to quantify compound-level QC variability and underpin compound confidence classification.

## When to use

Use when a targeted LC-MS peak table (tab-delimited with aliquot, compound, area, type, injection_time, batch columns) contains pooled QC samples (SQC/LQC) and you need a per-compound QC-reproducibility metric for compound filtering.

## When NOT to use

- When a compound is not detected in LQC or SQC samples it is assumed not present and will not be reported by mzQuality, so RSDQC cannot be computed for it
- Data that have not been peak-integrated to a compound-by-sample area table (mzQuality processes peak-integrated data independently of vendor software and does not process raw peaks like OpenMS or MZmine)
- When sample-level outliers, not compound QC reproducibility, are the target — the Rosner Test on QC samples is the corresponding sample-based check

## Inputs

- Tab-delimited peak-integrated text file with mandatory columns: aliquot, compound, area, type, injection_time, batch
- SummarizedExperiment object built by mzQuality's buildExperiment
- Pooled study quality control samples (SQC) and/or LQC samples identified via the `type` column
- Assigned internal standards per compound

## Outputs

- Per-compound RSDQC values stored in the assay/rowData slots of the SummarizedExperiment
- Recommended internal standard per compound (lowest RSDQC)
- Compound confidence classification into High confidence (RSDQC < 15%), With Caution (15–30%), and Low Signal to Noise (RSDQC > 30% or background e-score > 40%)
- Boolean `use` column in rowData indicating compounds reliable for reporting

## How to apply

Load the tab-delimited peak-integrated table with mzQuality's readData, which validates the mandatory columns (aliquot, compound, area, type, injection_time, batch), then build the SummarizedExperiment with buildExperiment and run doAnalysis. doAnalysis computes the ratio between each compound and its assigned internal standard, performs batch correction using the pooled study QC samples (SQC), and then calculates the RSD of QC samples per compound. Before computing RSDQC, exclude NA values (treated as absent data points and not used in calculations) and potential outliers flagged by mzQuality, which are automatically excluded from further calculations. The threshold for RSDQC is 30% per the metabolomics QA/QC consortium guidelines; compounds with RSDQC > 30% are flagged low quality, and RSDQC < 15% marks high confidence. RSDQC also drives internal-standard recommendation: mzQuality suggests the internal standard that yields the lowest RSDQC over all compound/IS combinations. Results are added to the assay and rowData slots, with a `use` column indicating whether each compound is reliable at the set thresholds.

## Related tools

- **mzQuality** (Computes compound/IS ratios, batch-corrects against pooled SQC samples, and calculates the RSD of QC samples (RSDQC) per compound for quality classification) — https://github.com/hankemeierlab/mzQuality
- **R** (Runtime environment (R ≥ 4.0) in which mzQuality is distributed and executed)
- **mzQualityDashboard** (Interactive Shiny front-end recommended for running the same RSDQC-based analysis without writing R code) — https://github.com/hankemeierlab/mzQualityDashboard

## Examples

```
path <- system.file("extdata", "example.tsv", package = "mzQuality"); exp <- doAnalysis(buildExperiment(readData(path)))
```

## Evaluation signals

- Every compound detected in QC samples receives a finite RSDQC value; NA values and mzQuality-flagged outliers are excluded from the calculation by construction
- RSDQC is computed on batch-corrected compound/IS ratios, not on raw areas
- Compounds with RSDQC > 30% must land in the Low Signal to Noise or With Caution tiers; RSDQC < 15% must be High confidence (provided the background e-score ≤ 40%)
- The mzQuality-recommended internal standard for a compound must be the candidate with the lowest RSDQC in the ranking table
- Calling doAnalysis twice overwrites previous values in the assay/rowData slots — verify no stale metrics remain

## Limitations

- Any NA value is treated as an absent data point and will not be used for the calculations
- Potential outliers flagged by mzQuality are automatically excluded from further calculations, so the reported RSDQC reflects the filtered set
- When a compound is not detected in LQC or SQC samples, it is assumed to not be present and will not be reported by mzQuality
- Thresholds are defaults (RSDQC 30%, e-score 40%) but are user-settable in doAnalysis, so outputs depend on the chosen cutoffs
- Calling doAnalysis again will overwrite previous values rather than appending

## Evidence

- [results] The threshold for RSDqc was set at 30% per the metabolomics QA/QC consortium, with a recommended background signal (e-score) value not exceeding 40%: "we set the threshold for the RSDqc at 30%. For the background signal, we recommend that the value of the e-score should not exceed 40%"
- [results] Compounds organized into three tabs by RSDQC thresholds: "Compounds are organized into three tabs based on quality and user-defined or default settings"
- [results] outliers excluded from calculations: "Potential outliers are flagged by mzQuality and are automatically excluded from further calculations"
- [results] NA treated as absent: "Any NA value is treated as an absent data point and will not be used for the calculations"
- [readme] doAnalysis computes ratio, batch correction, QC RSD, etc.: "Calculate the ratio between the compounds and assigned internal standards,
2.  Perform batch correction using the pooled study quality control samples (SQC)"
- [readme] Thresholds can be set in doAnalysis: [evidence span withheld — anchor status: external_doc]
- [readme] mzQuality recommends IS with lowest RSDQC: [evidence span withheld — anchor status: external_doc]
