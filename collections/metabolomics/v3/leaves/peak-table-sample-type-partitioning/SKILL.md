---
name: peak-table-sample-type-partitioning
description: Use when a tab-delimited peak-integrated table (columns aliquot, compound,
  area, type, injection_time, batch) from targeted LC-MS dMRM measurements must be
  split by sample type (QC, blank, study) so that per-compound RSD of QC area ratios
  and blank-derived background signal (e-score) can be.
when_to_use_negative:
- The input is not a peak-integrated table with the mandatory mzQuality columns (aliquot,
  compound, area, type, injection_time, batch) — readData will reject it
- The dataset is untargeted/feature-based (e.g. mzML features from OpenMS or MZmine)
  rather than peak-integrated targeted LC-MS dMRM output; mzQuality processes peak-integrated
  data independently of vendor software and is not a peak-picking tool
- The goal is batch correction or outlier detection rather than compound confidence-tier
  classification; those are separate mzQuality workflow steps
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: mzQuality
  role: Reads the tab-delimited peak-integrated table, partitions samples by the `type`
    column, computes RSDQC and background e-score, and assigns compounds to High Confidence
    / Caution / Low SNR tiers via `doAnalysis`.
  repo: https://github.com/hankemeierlab/mzQuality
- name: R
  role: Runtime environment in which the mzQuality package (version 4.0 or later)
    is installed and the SummarizedExperiment analysis is executed.
  repo: https://github.com/hankemeierlab/mzQuality
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1021/jasms.5c00073
    title: 'mzQuality: An Open-Source Software Tool for Quality Monitoring and Reporting
      of Targeted Mass Spectrometry Measurements'
  source_content_hash: sha256:e1222bace4c4ceeea281e79bbfda7497168c9735f64b9999a2e68f0c7f638c00
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/hankemeierlab/mzQuality
  tools:
  - R
  - mzQuality
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9411764705882353
n_claims: 17
---

# peak-table-sample-type-partitioning

## Summary

Partition a tab-delimited peak-integrated metabolomics table into QC (LQC/SQC), blank, and study sample groups using the mzQuality `type` column, then compute per-compound RSDQC and background e-score metrics and classify each compound into High Confidence, Caution, or Low SNR tiers using the RSDQC <15%, 15–30%, and RSDQC >30% / e-score >40% thresholds.

## When to use

Use when a tab-delimited peak-integrated table (columns aliquot, compound, area, type, injection_time, batch) from targeted LC-MS dMRM measurements must be split by sample type (QC, blank, study) so that per-compound RSD of QC area ratios and blank-derived background signal (e-score) can be computed and compounds classified into confidence tiers.

## When NOT to use

- The input is not a peak-integrated table with the mandatory mzQuality columns (aliquot, compound, area, type, injection_time, batch) — readData will reject it
- The dataset is untargeted/feature-based (e.g. mzML features from OpenMS or MZmine) rather than peak-integrated targeted LC-MS dMRM output; mzQuality processes peak-integrated data independently of vendor software and is not a peak-picking tool
- The goal is batch correction or outlier detection rather than compound confidence-tier classification; those are separate mzQuality workflow steps

## Inputs

- Tab-delimited peak-integrated text file with mandatory columns aliquot, compound, area, type, injection_time, and batch
- Optional SummarizedExperiment object built by mzQuality's buildExperiment
- Sample-type annotations in the `type` column distinguishing QC (LQC/SQC), blank, and study groups

## Outputs

- SummarizedExperiment with per-compound RSDQC and e-score values added to rowData
- A `use` boolean column in rowData flagging compounds reliable for reporting under the thresholds
- Compound-confidence-tier assignments (High Confidence, Caution, Low SNR) exportable as a CSV such as compound_confidence_classification.csv

## How to apply

Read the tab-delimited file with mzQuality's `readData`, which validates the mandatory columns aliquot, compound, area, type, injection_time, and batch, then build a SummarizedExperiment with `buildExperiment`. Partition samples by the `type` column into QC (LQC/SQC), blank, and study groups, treating any NA value as an absent data point that is excluded from calculations and omitting compounds not detected in LQC or SQC samples. Compute the per-compound relative standard deviation of area ratios across QC replicates (RSDQC) over the batches and estimate the per-compound background e-score from blank samples relative to measured study-sample signal. Assign confidence levels with the fixed thresholds: RSDqc < 15% = High Confidence, 15–30% = Caution, and RSDqc > 30% or e-score > 40% = Low SNR; run `doAnalysis` to compute all of these metrics, which will add them to the `rowData` of the experiment.

## Related tools

- **mzQuality** (Reads the tab-delimited peak-integrated table, partitions samples by the `type` column, computes RSDQC and background e-score, and assigns compounds to High Confidence / Caution / Low SNR tiers via `doAnalysis`.) — https://github.com/hankemeierlab/mzQuality
- **R** (Runtime environment in which the mzQuality package (version 4.0 or later) is installed and the SummarizedExperiment analysis is executed.) — https://github.com/hankemeierlab/mzQuality

## Examples

```
Rscript -e 'library(mzQuality); exp <- buildExperiment(readData("example.tsv")); exp <- doAnalysis(exp); exp <- exp[rowData(exp)$use, exp$use]'
```

## Evaluation signals

- Every compound is assigned exactly one of the three tiers (High Confidence, Caution, Low SNR) with no unassigned entries
- Compounds flagged Low SNR satisfy the threshold criteria: RSDQC > 30% or e-score > 40%
- RSDQC values for compounds in the High Confidence tier are strictly below 15% and Caution-tier compounds fall in the 15–30% range
- No compound that is not detected in LQC or SQC samples is reported in the classification output
- The `use` column in rowData is populated for every compound and reflects the RSDQC/background-signal/presence criteria

## Limitations

- Any NA value is treated as an absent data point and will not be used for the calculations, which can lower effective replicate counts
- When a compound is not detected in LQC or SQC samples, mzQuality assumes it is not present and will not report it
- Thresholds (RSDqc 30%, e-score 40%) are user-adjustable in doAnalysis; changing them changes tier assignments and must be documented
- The `use` flag can be manually overridden to TRUE or FALSE, so automated tier assignment may not equal the final curated selection

## Evidence

- [results] Three-tier classification thresholds: [evidence span withheld — anchor status: not_found]
- [results] Background signal threshold: "we set the threshold for the RSDqc at 30%. For the background signal, we recommend that the value of the e-score should not exceed 40%"
- [readme] readData mandatory column validation: [evidence span withheld — anchor status: external_doc]
- [readme] doAnalysis step list: [evidence span withheld — anchor status: external_doc]
- [results] NA handling: "Any NA value is treated as an absent data point and will not be used for the calculations"
- [results] Compound-not-in-QC handling: "When a compound is not detected in LQC or SQC samples, the compound will be assumed to not be present and will not be reported by mzQuality"
- [readme] use flag: [evidence span withheld — anchor status: external_doc]
