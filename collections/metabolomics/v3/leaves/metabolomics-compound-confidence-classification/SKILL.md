---
name: metabolomics-compound-confidence-classification
description: Use when a batch-corrected targeted metabolomics peak-integrated table
  must be screened per compound to decide which metabolites are reliable enough to
  report, flagging those with poor QC reproducibility or high blank background before
  downstream statistics.
when_to_use_negative:
- The input is raw vendor peak-processing output (mzML/MZmine/OpenMS results) rather
  than an already peak-integrated tab-delimited table — mzQuality processes peak-integrated
  data independently of vendor software and expects the aliquot/compound/area/type/injection_time/batch
  schema
- A compound is not detected in any LQC or SQC sample — mzQuality assumes it is not
  present and does not report it, so no confidence tier can be assigned
- The goal is feature-level untargeted QC (e.g. baseline noise filtering or retention
  time alignment) rather than per-compound confidence classification of targeted metabolites
edam_operation: http://edamontology.org/operation_3218
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: mzQuality
  role: R package that computes RSDQC, background e-score, and assigns the three confidence
    tiers via doAnalysis, exposing the decision through the rowData `use` column
  repo: https://github.com/hankemeierlab/mzQuality
- name: mzQualityDashboard
  role: Shiny front-end recommended for interactive inspection of the compound confidence
    tabs (High confidence / With Caution / Low SNR) produced by the classification
  repo: https://github.com/hankemeierlab/mzQualityDashboard
- name: R
  role: Runtime for mzQuality (version 4.0 or later required); classification results
    are exported as R-compatible data ready for statistical analysis
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1021/jasms.5c00073
    title: 'mzQuality: An Open-Source Software Tool for Quality Monitoring and Reporting
      of Targeted Mass Spectrometry Measurements'
  source_content_hash: sha256:95bcdc5bcbe46ee42b2743088ff63562c13b9828d5a91cc8c8852421e17a0454
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/hankemeierlab/mzQuality
  tools:
  - R
  - mzQuality
  - mzQualityDashboard
  edam_operation: http://edamontology.org/operation_3218
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8666666666666667
n_claims: 15
---

# metabolomics-compound-confidence-classification

## Summary

Classify targeted LC-MS compounds into High confidence, With Caution, or Low Signal to Noise tiers using mzQuality's per-compound RSDQC (relative standard deviation across QC samples) and blank-sample background e-score, with default cutoffs at RSDQC 15%/30% and e-score 40%.

## When to use

Use when a batch-corrected targeted metabolomics peak-integrated table must be screened per compound to decide which metabolites are reliable enough to report, flagging those with poor QC reproducibility or high blank background before downstream statistics.

## When NOT to use

- The input is raw vendor peak-processing output (mzML/MZmine/OpenMS results) rather than an already peak-integrated tab-delimited table — mzQuality processes peak-integrated data independently of vendor software and expects the aliquot/compound/area/type/injection_time/batch schema
- A compound is not detected in any LQC or SQC sample — mzQuality assumes it is not present and does not report it, so no confidence tier can be assigned
- The goal is feature-level untargeted QC (e.g. baseline noise filtering or retention time alignment) rather than per-compound confidence classification of targeted metabolites

## Inputs

- Tab-delimited peak-integrated table with mandatory columns aliquot, compound, area, type, injection_time, and batch (e.g. SCIEX OS-integrated dMRM output)
- SummarizedExperiment object built via buildExperiment from readData
- Sample `type` column values distinguishing study samples, pooled study QC (SQC), LQC, and blank samples

## Outputs

- Per-compound RSDQC values and background e-scores added to rowData of the SummarizedExperiment
- Confidence-level assignment per compound: High confidence (RSDqc <15%), With Caution (15–30%), or Low Signal to Noise (RSDqc >30% or e-score >40%)
- `use` boolean column in rowData and colData indicating whether each compound/sample is reliable for reporting
- Subset experiment filtered to recommended compounds and samples: exp[rowData(exp)$use, exp$use]

## How to apply

Load the tab-delimited peak-integrated table (columns: aliquot, compound, area, type, injection_time, batch) with mzQuality's readData, build the SummarizedExperiment object with buildExperiment, then run doAnalysis to compute the compound/IS ratio, SQC-based batch correction, background signal percentage, and RSD of QC samples. Partition samples by the type column into QC (LQC/SQC), blank, and study groups; treat any NA value as an absent data point excluded from all calculations, and drop compounds not detected in LQC or SQC samples entirely. Compute per-compound RSDQC (RSD of area ratios across QC replicates over six batches) and the background e-score from blanks relative to measured signal, then classify: RSDQC < 15% = High confidence, 15–30% = With Caution, and RSDQC > 30% or e-score > 40% = Low Signal to Noise. The rationale follows the metabolomics quality assurance consortium's guideline of RSDqc ≤ 30%, with the 40% e-score limit guarding against compounds whose signal is dominated by background. mzQuality writes a `use` TRUE/FALSE column into rowData reflecting the combined classification, and users may manually override individual compounds before subsetting.

## Related tools

- **mzQuality** (R package that computes RSDQC, background e-score, and assigns the three confidence tiers via doAnalysis, exposing the decision through the rowData `use` column) — https://github.com/hankemeierlab/mzQuality
- **mzQualityDashboard** (Shiny front-end recommended for interactive inspection of the compound confidence tabs (High confidence / With Caution / Low SNR) produced by the classification) — https://github.com/hankemeierlab/mzQualityDashboard
- **R** (Runtime for mzQuality (version 4.0 or later required); classification results are exported as R-compatible data ready for statistical analysis)

## Examples

```
exp <- doAnalysis(buildExperiment(readData(path))); exp <- exp[rowData(exp)$use, exp$use]
```

## Evaluation signals

- Every reported compound is assigned exactly one of the three confidence tiers (High confidence, With Caution, Low Signal to Noise) — no compound is left unclassified
- All compounds flagged Low Signal to Noise satisfy the threshold criteria: RSDQC > 30% or background e-score > 40%
- Compounds with High confidence tier have RSDqc strictly <15%; With Caution compounds fall in 15–30%
- NA values are excluded from calculations (no data point shown in individual compound plots) and compounds undetected in LQC/SQC are absent from the output
- rowdata `use` column is TRUE only for compounds passing the RSDQC, background-signal, and QC-presence combination of thresholds

## Limitations

- Compounds not detected in LQC or SQC samples are assumed not present and are not reported by mzQuality at all
- Any NA value is treated as an absent data point and will not be used for the calculations
- Thresholds (RSDQC 15%/30%, e-score 40%) are user-adjustable defaults; changing them in doAnalysis changes which compounds land in each tier, and the automatic `use` selection can be overridden manually
- The e-score/background-signal assessment depends on blank samples being included in the input; without blanks the background cutoff cannot be applied
- Example dataset spans six batches (419 CoSTREAM EDTA plasma study samples); classification performance may differ for studies with fewer batches or QC samples

## Evidence

- [results] Three-tier classification thresholds: [evidence span withheld — anchor status: not_found]
- [results] Threshold rationale per QA/QC consortium: "we set the threshold for the RSDqc at 30%. For the background signal, we recommend that the value of the e-score should not exceed 40%"
- [intro] Compound flagging criterion: "We defined the threshold at a relative standard deviation of QC > 30%, which follows the guidelines of the metabolomics quality assurance"
- [results] NA handling and QC presence: "When a compound is not detected in LQC or SQC samples, the compound will be assumed to not be present and will not be reported by mzQuality"
- [results] NA treated as absent: "Any NA value is treated as an absent data point and will not be used for the calculations"
- [readme] Compound selection basis in README: [evidence span withheld — anchor status: external_doc]
- [readme] Input requirements and readData/buildExperiment: [evidence span withheld — anchor status: external_doc]
- [readme] doAnalysis computes QC ratio and background: [evidence span withheld — anchor status: external_doc]
- [readme] High/Caution/Low SNR reporting tiers in README: [evidence span withheld — anchor status: external_doc]
