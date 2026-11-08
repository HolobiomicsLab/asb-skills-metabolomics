---
name: metabolomics-violin-plot-visualization
description: 'Use when quality-controlling targeted LC-MS peak-area data: comparing
  per-batch distributions of QC samples, or evaluating pre- vs post-batch-correction
  variation before relying on corrected ratios.'
when_to_use_negative:
- You need batch-overall sample clustering rather than per-sample-type distribution
  inspection — use pcaPlot (e.g. on ratio_corrected) instead.
- The data is untargeted/peak-picking output rather than peak-integrated targeted
  LC-MS measurements that mzQuality processes.
- Compounds are undetected in LQC/SQC samples — mzQuality assumes they are absent
  and will not report them, so no violin distribution can be assessed.
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: mzQuality
  role: R package providing the violinPlot function that renders per-sample-type distribution
    plots for compound areas, ratios, and batch-corrected ratios
  repo: https://github.com/hankemeierlab/mzQuality
- name: mzQualityDashboard
  role: Shiny dashboard wrapping mzQuality's plots (including violin plots) for interactive
    inspection without scripting
  repo: https://github.com/hankemeierlab/mzQualityDashboard
- name: R
  role: runtime environment; mzQuality requires R version 4.0 or later and produces
    R-compatible outputs
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1021/jasms.5c00073
    title: 'mzQuality: An Open-Source Software Tool for Quality Monitoring and Reporting
      of Targeted Mass Spectrometry Measurements'
  source_content_hash: sha256:2c9453a0a5c211a7f0a13acd125e058c34465f4eeb42cc042aa7cd10eaf61378
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/hankemeierlab/mzQuality
  tools:
  - R
  - R-Shiny
  - mzQuality
  - mzQualityDashboard
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# metabolomics-violin-plot-visualization

## Summary

Visualize per-batch variation in targeted mass spectrometry metabolomics data with mzQuality's violin plots, which show the distribution of compound or internal standard signals for a given sample type (e.g. LQC/SQC) so batch effects and QC spread can be inspected before and after between-batch correction.

## When to use

Use when quality-controlling targeted LC-MS peak-area data: comparing per-batch distributions of QC samples, or evaluating pre- vs post-batch-correction variation before relying on corrected ratios.

## When NOT to use

- You need batch-overall sample clustering rather than per-sample-type distribution inspection — use pcaPlot (e.g. on ratio_corrected) instead.
- The data is untargeted/peak-picking output rather than peak-integrated targeted LC-MS measurements that mzQuality processes.
- Compounds are undetected in LQC/SQC samples — mzQuality assumes they are absent and will not report them, so no violin distribution can be assessed.

## Inputs

- tab-delimited peak-area table with compound areas, internal standard areas, and sample metadata (mandatory columns checked by readData)
- SummarizedExperiment object built with buildExperiment
- sample-type annotations distinguishing study samples, LQC, and SQC
- assay selection: raw areas, compound/IS 'ratio', or batch-corrected 'ratio_corrected'

## Outputs

- violin plots showing per-batch distributions for a given sample type
- side-by-side pre- vs post-batch-correction distribution panels
- visual identification of samples with abnormal total signal (e.g. low median area) and batch-boundary deviations
- supporting evidence alongside compoundPlot and aliquotPlot diagnostics in HTML/Excel reports from createReports

## How to apply

Read the tab-delimited peak-area table with mzQuality's `readData`, build the SummarizedExperiment with `buildExperiment`, and run `doAnalysis` so the compound/IS ratio and the SQC-based batch correction (`ratio_corrected`) are available. For each sample type (LQC, SQC, study samples), generate a violin plot with `violinPlot(exp)`, plotting the per-batch distributions either of raw compound areas or of the corrected compound/IS ratios. Compare distributions across batches — for the example dataset (105 compounds, 584 samples over six batches), deviations such as a spike in the median ratio at the end of batch 4 / start of batch 5, or a QC sample with unusually low total signal, appear as visible distribution shifts or shape changes in the violin plot. Note that mzQuality flags samples failing the Rosner outlier test and compounds failing the RSDQC ≤ 30% / e-score ≤ 40% thresholds; you may exclude flagged points and re-render violins to assess whether flagged variation is corrected. Validate by confirming that post-correction SQC distributions are aligned and centered within the study samples, ideally corroborated by an overlapping-ellipses PCA from `pcaPlot` and reduced QC RSD.

## Related tools

- **mzQuality** (R package providing the violinPlot function that renders per-sample-type distribution plots for compound areas, ratios, and batch-corrected ratios) — https://github.com/hankemeierlab/mzQuality
- **mzQualityDashboard** (Shiny dashboard wrapping mzQuality's plots (including violin plots) for interactive inspection without scripting) — https://github.com/hankemeierlab/mzQualityDashboard
- **R** (runtime environment; mzQuality requires R version 4.0 or later and produces R-compatible outputs)

## Examples

```
path <- system.file("extdata","example.tsv",package="mzQuality"); exp <- doAnalysis(buildExperiment(readData(path))); violinPlot(exp)
```

## Evaluation signals

- Post-correction SQC violin distributions are aligned and centered within the study samples, matching the overlapping-ellipses PCA expectation for the six-batch example
- Deviations observed pre-correction (e.g. last two samples of batch 4 and first three of batch 5 median-ratio shift) are visibly reduced in the post-correction violins
- Outliers flagged by the Rosner test on QC samples can be excluded from plots; flagged points should not drive interpretation
- Only compounds meeting the QC thresholds (RSDqc ≤ 30%, e-score ≤ 40%) are assessed; use rowData$use to filter
- Every plotted distribution respects missing values: samples with NA show no data point and NAs are excluded from calculations

## Limitations

- NA values are treated as absent data points and produce no data point in the plot; they are excluded from calculations
- When a compound is not detected in LQC or SQC samples it is assumed absent and not reported, so it cannot be visualized
- mzQuality does not itself process raw peak data — peak integration must be done upstream (e.g. SCIEX OS dMRM integration in the paper)
- Flagged outliers are automatically excluded from further calculations, which may hide underlying data issues if not reviewed
- An R-compatible output is produced, but interactive use is recommended via the mzQualityDashboard rather than raw plotting calls

## Evidence

- [abstract] mzQuality visualizes data through sample plots, PCA plots, and violin plots: "mzQuality visualizes data through sample plots, PCA plots, and violin plots"
- [readme] violinPlot(exp) function from README: "Violinplot showing the distribution for a given sample type
violinPlot(exp)"
- [results] PCA post-correction alignment claim: [evidence span withheld — anchor status: not_found]
- [results] batch-boundary deviation example: "Deviations in the median ratio are observed in the last two samples of batch 4 and the first three samples of batch 5"
- [intro] RSDqc threshold definition: "We defined the threshold at a relative standard deviation of QC > 30%, which follows the guidelines of the metabolomics quality assurance"
