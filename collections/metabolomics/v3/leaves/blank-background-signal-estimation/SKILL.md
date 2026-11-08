---
name: blank-background-signal-estimation
description: Use when targeted LC-MS metabolomics peak-integrated tables (with aliquot,
  compound, area, type, injection_time, batch columns) include blank samples and you
  must quantify per-compound background contamination to decide compound reliability.
when_to_use_negative:
- The dataset contains no blank samples — the e-score cannot be computed and the background
  filter is not applicable.
- You need de novo peak detection or baseline noise filtering from raw vendor files;
  mzQuality operates on already peak-integrated data and processes peak-integrated
  data independently of vendor software.
- The goal is untargeted feature finding (e.g. OpenMS or MZmine-style peak picking);
  this skill is a post-integration reliability filter on targeted MRM tables.
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: mzQuality
  role: Reads the peak-integrated table, computes the per-compound background signal
    percentage (e-score) from blank versus study samples inside doAnalysis, and flags
    compounds above the 40% background threshold as low signal-to-noise.
  repo: https://github.com/hankemeierlab/mzQuality
- name: mzQualityDashboard
  role: Shiny interface built on mzQuality for interactively inspecting background-signal
    results and the resulting High Confidence / Caution / Low SNR compound tabs.
  repo: https://github.com/hankemeierlab/mzQualityDashboard
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1021/jasms.5c00073
    title: 'mzQuality: An Open-Source Software Tool for Quality Monitoring and Reporting
      of Targeted Mass Spectrometry Measurements'
  source_content_hash: sha256:4682867a0171dcd880959884e14952065a272acd27e341556c50094a4e426297
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
yes_rate: 0.7777777777777778
n_claims: 9
---

# Blank background signal estimation

## Summary

Estimates each compound's background signal (e-score) as the percentage of blank-sample signal relative to measured study-sample signal in mzQuality, and uses the resulting value to flag compounds whose blank contribution exceeds the 40% reliability threshold.

## When to use

Use when targeted LC-MS metabolomics peak-integrated tables (with aliquot, compound, area, type, injection_time, batch columns) include blank samples and you must quantify per-compound background contamination to decide compound reliability.

## When NOT to use

- The dataset contains no blank samples — the e-score cannot be computed and the background filter is not applicable.
- You need de novo peak detection or baseline noise filtering from raw vendor files; mzQuality operates on already peak-integrated data and processes peak-integrated data independently of vendor software.
- The goal is untargeted feature finding (e.g. OpenMS or MZmine-style peak picking); this skill is a post-integration reliability filter on targeted MRM tables.

## Inputs

- Tab-delimited peak-integrated targeted LC-MS table with mandatory columns: aliquot, compound, area, type, injection_time, batch (e.g. from SCIEX OS 2.1.6.59781 dMRM integration)
- Sample-type annotations distinguishing blank, LQC/SQC quality control, and study samples in the `type` column

## Outputs

- Per-compound background signal percentage (e-score) added to the SummarizedExperiment rowData/assay slots
- Confidence-level classification of compounds into High Confidence, Caution, and Low Signal to Noise tabs based on e-score and RSDQC thresholds
- Boolean `use` flags in rowData marking compounds reliable for reporting given the set thresholds
- HTML/tab-delimited/Excel report files from createReports containing background and confidence tables

## How to apply

Load the tab-delimited peak-integrated table with mzQuality's `readData`, which verifies mandatory columns (aliquot, compound, area, type, injection_time, batch), then build the SummarizedExperiment with `buildExperiment` and run `doAnalysis`, whose third step 'Calculate the percentage of background signal compared to the study samples' computes the per-compound e-score from blank samples versus study samples. Treat NA values as absent data points that are excluded from calculations, and treat compounds not detected in LQC or SQC samples as not present and not reported. Classify compounds using the e-score threshold: values not exceeding 40% are acceptable background; e-score > 40% together with RSDQC > 30% places a compound in the 'Low Signal to Noise' category flagged for removal. The background metric feeds the `use` column in `rowData` that marks compounds reliable for reporting, and can be inspected alongside RSDQC in the High Confidence / Caution / Low SNR tabs of the report.

## Related tools

- **mzQuality** (Reads the peak-integrated table, computes the per-compound background signal percentage (e-score) from blank versus study samples inside doAnalysis, and flags compounds above the 40% background threshold as low signal-to-noise.) — https://github.com/hankemeierlab/mzQuality
- **mzQualityDashboard** (Shiny interface built on mzQuality for interactively inspecting background-signal results and the resulting High Confidence / Caution / Low SNR compound tabs.) — https://github.com/hankemeierlab/mzQualityDashboard

## Examples

```
exp <- buildExperiment(readData(path)); exp <- doAnalysis(exp); exp <- exp[rowData(exp)$use, exp$use]
```

## Evaluation signals

- Every reported compound satisfies the threshold invariant: e-score ≤ 40% OR (e-score > 40% only when categorized as Low Signal to Noise and RSDQC > 30%).
- Each compound appears in exactly one of the three quality tabs (High confidence with RSDqc < 15%, With Caution with 15–30%, Low Signal to Noise with background > 40% or RSDqc > 30%).
- Compounds not detected in LQC or SQC samples are absent from the output, consistent with mzQuality treating them as not present.
- NA entries in the input table contribute no background datapoint (treated as absent, not as zero signal).
- The computed e-score values match the background-signal assay written to rowData after doAnalysis, and createReports' Low SNR tab lists exactly the compounds exceeding the background or RSDQC thresholds.

## Limitations

- The e-score is computed only from samples annotated as blanks via the `type` column; mislabeled blanks distort background estimates.
- Any NA value is treated as an absent data point and excluded from calculations, so sparse blank data may under-represent true background.
- Compounds not detected in LQC or SQC samples are assumed absent and are not reported at all, so no e-score is produced for them.
- Thresholds (40% background, 30% RSDQC) are user-configurable in doAnalysis; changing them alters which compounds are flagged and requires re-running the analysis.

## Evidence

- [abstract] Background signal metric definition: "Compound quality is further assessed by evaluating the relative standard deviation of quality control samples and the background signal from blank samples."
- [results] Threshold for background signal: "For the background signal, we recommend that the value of the e-score should not exceed 40%"
- [results] Confidence tab thresholds including background: [evidence span withheld — anchor status: not_found]
- [readme] doAnalysis step that computes background: [evidence span withheld — anchor status: external_doc]
- [results] NA handling rule: "Any NA value is treated as an absent data point and will not be used for the calculations"
- [results] Absent compounds not reported: "When a compound is not detected in LQC or SQC samples, the compound will be assumed to not be present and will not be reported by mzQuality"
