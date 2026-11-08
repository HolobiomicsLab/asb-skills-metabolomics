---
name: qc-threshold-based-compound-flagging
description: Use when a peak-integrated targeted LC-MS table (with aliquot, compound,
  area, type, batch columns) has been batch-corrected via pooled SQC samples and you
  must decide which compounds are reliable enough to report.
when_to_use_negative:
- Input is raw LC-MS vendor peak data that has not yet been peak-integrated to a compound
  x sample table (mzQuality does not process raw peaks; OpenMS/MZmine handle that
  step).
- The dataset has no QC or blank sample type annotations, since RSDqc and background
  e-score cannot be computed without pooled QC and blank samples.
- Compounds lack stable-isotope internal standard assignments and calibration concentrations,
  since ratio-based RSD and e-score are meaningless in that configuration.
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: mzQuality
  role: Computes RSDqc and background e-score per compound and assigns compound reliability
    tiers via threshold-based flagging in doAnalysis; also exposes the use column
    for manual override.
  repo: https://github.com/hankemeierlab/mzQuality
- name: R
  role: Runtime environment for mzQuality (requires R 4.0 or later) executing the
    threshold-based flagging logic.
- name: mzQualityDashboard
  role: Shiny dashboard built on mzQuality that visualizes the tiered compound classifications
    (High Confidence, Caution, Low SNR) for interactive inspection.
  repo: https://github.com/hankemeierlab/mzQualityDashboard
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1021/jasms.5c00073
    title: 'mzQuality: An Open-Source Software Tool for Quality Monitoring and Reporting
      of Targeted Mass Spectrometry Measurements'
  source_content_hash: sha256:46412800ac0a31ec08ff8e9c6867329cf506afdb0c5f8ce41bffa0cdf03596ec
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/hankemeierlab/mzQuality
  tools:
  - R
  - mzQuality
  - mzQualityDashboard
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# qc-threshold-based-compound-flagging

## Summary

Classifies targeted metabolomics compounds into quality tiers (High confidence, Caution, Low Signal-to-Noise) using mzQuality thresholds on QC-sample relative standard deviation (RSDqc) and blank-sample background e-score, so unreliable compounds are excluded from reporting.

## When to use

Use when a peak-integrated targeted LC-MS table (with aliquot, compound, area, type, batch columns) has been batch-corrected via pooled SQC samples and you must decide which compounds are reliable enough to report.

## When NOT to use

- Input is raw LC-MS vendor peak data that has not yet been peak-integrated to a compound x sample table (mzQuality does not process raw peaks; OpenMS/MZmine handle that step).
- The dataset has no QC or blank sample type annotations, since RSDqc and background e-score cannot be computed without pooled QC and blank samples.
- Compounds lack stable-isotope internal standard assignments and calibration concentrations, since ratio-based RSD and e-score are meaningless in that configuration.

## Inputs

- Tab-delimited peak-integrated table with mandatory columns aliquot, compound, area, type, injection_time, batch (e.g. SCIEX OS export)
- Optional known concentrations for calibration lines
- SummarizedExperiment object from mzQuality's buildExperiment

## Outputs

- Per-compound reliability flag (use=TRUE/FALSE in rowData)
- Compound-to-tier mapping: High confidence / Caution / Low Signal to Noise
- Per-compound RSDqc and background e-score values
- Exclusion list of compounds flagged Low Signal to Noise or absent from LQC/SQC samples

## How to apply

Load the tab-delimited peak-integrated output (from e.g. SCIEX OS) with mzQuality's readData, which checks the mandatory columns (aliquot, compound, area, type, batch), then build the SummarizedExperiment object with buildExperiment. Run doAnalysis to compute compound/IS ratios, SQC-based batch correction, background signal as percentage of study-sample signal, and RSD of QC samples. mzQuality flags compounds as unreliable based on the combination of RSDqc, background signal percentage, and presence in QC samples; default thresholds are RSDqc > 30% (following metabolomics QA/QC consortium guidelines) and a background e-score not exceeding 40%. Results populate three tiers: High confidence (RSDqc <15%), With Caution (RSDqc 15–30%), and Low Signal to Noise (e-score >40% or RSDqc >30%). The `use` column in rowData marks TRUE/FALSE reliability per compound, which can be manually overridden; unreliable compounds are excluded from further calculations and should not appear in downstream statistical analysis. Validation: every compound should be assigned exactly one tier, and Low Signal to Noise compounds must satisfy the threshold criteria (RSDqc >30% or e-score >40%).

## Related tools

- **mzQuality** (Computes RSDqc and background e-score per compound and assigns compound reliability tiers via threshold-based flagging in doAnalysis; also exposes the use column for manual override.) — https://github.com/hankemeierlab/mzQuality
- **R** (Runtime environment for mzQuality (requires R 4.0 or later) executing the threshold-based flagging logic.)
- **mzQualityDashboard** (Shiny dashboard built on mzQuality that visualizes the tiered compound classifications (High Confidence, Caution, Low SNR) for interactive inspection.) — https://github.com/hankemeierlab/mzQualityDashboard

## Examples

```
exp <- buildExperiment(readData(path)); exp <- doAnalysis(exp); exp <- exp[rowData(exp)$use, exp$use]
```

## Evaluation signals

- Every compound appears in exactly one of the three tiers (High confidence, With Caution, Low Signal to Noise) with no unassigned entries.
- Compounds flagged Low Signal to Noise satisfy RSDqc > 30% or background e-score > 40%.
- The `use` column in rowData is populated as TRUE/FALSE and no compound is left NA/NULL before subsetting the experiment.
- Compounds not detected in LQC or SQC samples are absent from the reported output.
- Re-running doAnalysis on the same input reproduces the same tier assignments (idempotency, since doAnalysis overwrites previous values).

## Limitations

- When a compound is not detected in LQC or SQC samples it is assumed absent and not reported at all, so it never receives a tier label.
- Any NA value is treated as an absent data point and excluded from calculations, which can skew RSDqc for sparse compounds.
- Thresholds (RSDqc 30%, e-score 40%) are the authors' defaults following QA/QC consortium guidelines; the dashboard exposes user-defined settings, but extreme thresholds may empty a tier.
- The `use` flag can be manually overridden, so blind trust in a saved rowData without inspecting overrides is unsafe.

## Evidence

- [abstract] Compound quality is assessed via the relative standard deviation of QC samples and blank-sample background signal, and compounds are classified into confidence levels based on these metrics.: "Compound quality is further assessed by evaluating the relative standard deviation of quality control samples and the background signal from blank samples."
- [intro] mzQuality flags compounds as low quality using a relative standard deviation of QC > 30% threshold and a background signal limit of no higher than 40%: "We defined the threshold at a relative standard deviation of QC > 30%, which follows the guidelines of the metabolomics quality assurance"
- [results] Compounds are organized into three tabs based on quality: High confidence (RSDqc <15%), With Caution (15–30%), Low Signal to Noise (background signal >40% or RSDqc >30%): "Compounds are organized into three tabs based on quality and user-defined or default settings"
- [results] The threshold for RSDqc was set at 30% per the metabolomics QA/QC consortium, with a recommended background signal (e-score) value not exceeding 40%: "we set the threshold for the RSDqc at 30%. For the background signal, we recommend that the value of the e-score should not exceed 40%"
- [results] When a compound is not detected in LQC or SQC samples, it is assumed not present and not reported by mzQuality: "When a compound is not detected in LQC or SQC samples, the compound will be assumed to not be present and will not be reported by mzQuality"
- [readme] mzQuality bases compound reliability decisions on the combination of RSDQC, background signal percentage, and presence in QC samples; use column in rowData and colData marks TRUE/FALSE: [evidence span withheld — anchor status: external_doc]
- [readme] mzQuality adds a use column in rowData/colData indicating whether compound/sample is reliable for reporting; values can be manually set to TRUE or FALSE to override automatic selection.: [evidence span withheld — anchor status: external_doc]
