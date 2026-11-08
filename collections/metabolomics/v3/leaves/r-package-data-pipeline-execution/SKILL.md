---
name: r-package-data-pipeline-execution
description: Use when a task requires reproducing mzQuality's internal-standard recommendation
  in R from peak-integrated targeted LC-MS data, or validating that the recommended
  IS is the one minimizing QC-sample variability (RSDQC).
when_to_use_negative:
- Input is raw, un-integrated LC-MS vendor data — mzQuality works only on peak-integrated
  output (e.g. from SCIEX OS) and does not perform peak integration itself
- Data lacks one or more mandatory columns (aliquot, compound, area, type, injection_time,
  batch) — readData will fail column validation, so reformat the table first
- Goal is interactive compound/sample inspection rather than programmatic IS-selection
  reproduction — the mzQualityDashboard Shiny app is recommended for that use
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: mzQuality
  role: R package whose readData, buildExperiment and doAnalysis functions execute
    the ratio/batch-correction/IS-suggestion pipeline being reproduced
  repo: https://github.com/hankemeierlab/mzQuality
- name: R
  role: Runtime environment (R ≥ 4.0 required) in which the mzQuality package pipeline
    is executed programmatically
- name: R-Shiny
  role: Framework behind the mzQualityDashboard front-end; alternative interactive
    route to the same analysis
  repo: https://github.com/hankemeierlab/mzQualityDashboard
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/jasms.5c00073
    title: 'mzQuality: An Open-Source Software Tool for Quality Monitoring and Reporting
      of Targeted Mass Spectrometry Measurements'
  source_content_hash: sha256:c816a57c5860d4d90be930a326c98d12e878714d17362e1c84ecbd16f5057a27
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/hankemeierlab/mzQuality
  tools:
  - R
  - R-Shiny
  - mzQuality
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# r-package-data-pipeline-execution

## Summary

Runs the mzQuality R package pipeline (readData → buildExperiment → doAnalysis) on tab-delimited peak-area data to reproduce, computationally, which internal standard mzQuality recommends for each compound — the candidate whose batch-corrected compound/IS ratios give the lowest RSD of QC samples (RSDQC).

## When to use

Use when a task requires reproducing mzQuality's internal-standard recommendation in R from peak-integrated targeted LC-MS data, or validating that the recommended IS is the one minimizing QC-sample variability (RSDQC).

## When NOT to use

- Input is raw, un-integrated LC-MS vendor data — mzQuality works only on peak-integrated output (e.g. from SCIEX OS) and does not perform peak integration itself
- Data lacks one or more mandatory columns (aliquot, compound, area, type, injection_time, batch) — readData will fail column validation, so reformat the table first
- Goal is interactive compound/sample inspection rather than programmatic IS-selection reproduction — the mzQualityDashboard Shiny app is recommended for that use

## Inputs

- Tab-delimited peak-integrated measurement file (e.g. inst/extdata/example.tsv) with mandatory columns: aliquot, compound, area, type, injection_time, batch
- SummarizedExperiment object produced by readData + buildExperiment
- Optional: known concentrations of calibration lines for downstream concentration/R² calculation

## Outputs

- Ranking of internal standard candidates by RSDQC of batch-corrected compound/IS ratios (is_selection_rsdqc.csv)
- Identified recommended internal standard per compound (lowest RSDQC, within ≤ 30% threshold)
- Updated assay/rowData/colData slots of the SummarizedExperiment containing ratios, corrected ratios, and QC metrics

## How to apply

Load the tab-delimited example or user data with mzQuality's readData function, which verifies that the mandatory columns (aliquot, compound, area, type, injection_time, batch) are present, then buildExperiment converts it into the SummarizedExperiment object used for all calculations. Apply doAnalysis, which computes compound/internal-standard ratios, performs between-batch correction on the pooled study QC (SQC) samples, and 'Suggests Internal Standards based on the calculated values'. To reproduce the recommendation exhaustively, exclude flagged potential outliers and NA values (treated as absent data points) from QC calculations, then for each candidate IS compute the relative standard deviation of QC samples (RSDQC) of the batch-corrected ratios and rank candidates ascending by RSDQC. Validate against mzQuality's thresholds: RSDqc ≤ 30% marks acceptable quality and < 15% corresponds to the High-confidence class, so the recommended IS should be the lowest-RSDQC candidate and should fall within the ≤ 30% acceptance band.

## Related tools

- **mzQuality** (R package whose readData, buildExperiment and doAnalysis functions execute the ratio/batch-correction/IS-suggestion pipeline being reproduced) — https://github.com/hankemeierlab/mzQuality
- **R** (Runtime environment (R ≥ 4.0 required) in which the mzQuality package pipeline is executed programmatically)
- **R-Shiny** (Framework behind the mzQualityDashboard front-end; alternative interactive route to the same analysis) — https://github.com/hankemeierlab/mzQualityDashboard

## Examples

```
path <- system.file("extdata", "example.tsv", package = "mzQuality"); exp <- buildExperiment(readData(path)); exp <- doAnalysis(exp = exp)
```

## Evaluation signals

- readData runs without column-validation errors: aliquot, compound, area, type, injection_time and batch are all present and correctly formatted
- The mzQuality-suggested internal standard coincides with the candidate having the lowest RSDQC after exhaustive per-IS calculation on batch-corrected ratios
- RSDQC of the recommended internal standard is ≤ 30% (mzQuality acceptance threshold); values < 15% correspond to the High-confidence tab
- Potential outliers flagged via the Rosner test on QC samples and NA values are excluded from the RSDQC computation before ranking
- Re-running doAnalysis reproduces consistent values stored in the assay/rowData/colData slots of the SummarizedExperiment

## Limitations

- Any NA value is treated as an absent data point, and compounds not detected in LQC or SQC samples are assumed not present and not reported — exhaustive RSDQC may therefore have unequal candidate coverage per compound
- Automatic compound/sample selection depends on user-set thresholds (RSDQC, background-signal percentage, presence in QC); the `use` column can be overridden manually
- Calling doAnalysis a second time overwrites previous values in the assay, rowData and colData slots — intermediate states are not preserved

## Evidence

- [readme] readData validates mandatory columns and formats: [evidence span withheld — anchor status: external_doc]
- [readme] buildExperiment creates the analysis object: [evidence span withheld — anchor status: external_doc]
- [readme] doAnalysis steps include ratio computation, SQC batch correction, and IS suggestion: "Calculate the ratio between the compounds and assigned internal standards,
2.  Perform batch correction using the pooled study quality control samples (SQC)"
- [readme] IS suggestion is one of the doAnalysis outputs: "Calculate the median area of the compounds in the samples,
8.  Suggest Internal Standards based on the calculated values."
- [results] RSDqc threshold of 30% set per metabolomics QA/QC consortium: "we set the threshold for the RSDqc at 30%. For the background signal, we recommend that the value of the e-score should not exceed 40%"
- [results] Quality classes defined by RSDqc bands: [evidence span withheld — anchor status: not_found]
- [results] Flagged outliers are excluded from calculations: "Potential outliers are flagged by mzQuality and are automatically excluded from further calculations"
- [results] NA values are dropped from calculations: "Any NA value is treated as an absent data point and will not be used for the calculations"
