---
name: lc-ms-qc-sample-pca-visualization
description: Use when a targeted LC-MS metabolomics study was measured in multiple
  batches and you need to verify, via PCA of sample types and batches, whether pooled
  QC (SQC) samples become aligned and centered across batches after batch correction.
when_to_use_negative:
- For untargeted metabolomics peak tables processed with tools like OpenMS or MZmine
  rather than mzQuality's SQC-based SummarizedExperiment workflow — this skill depends
  on mzQuality's batch-corrected 'ratio_corrected' assay and sample-type annotations.
- When you only need per-compound QC relative standard deviation (RSD) checks rather
  than a multivariate view of between-batch alignment; the RSDqc <= 30% threshold
  tabulation is a separate evaluation.
- When samples are not measured in multiple batches with pooled QC (SQC) samples,
  since the PCA's purpose here is comparing SQC vs study samples across batches.
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: mzQuality
  role: R package whose pcaPlot function generates the PCA plot of aliquot distribution
    per type and batch for batch-corrected ratios; its doAnalysis produces the batch-corrected
    assay used as PCA input.
  repo: https://github.com/hankemeierlab/mzQuality
- name: R
  role: Runtime environment for the mzQuality package (R 4.0 or later required) in
    which pcaPlot and violinPlot are executed.
- name: R-Shiny
  role: Framework behind the mzQualityDashboard interface, recommended for interactive
    use of these QC visualizations by new R users.
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1021/jasms.5c00073
    title: 'mzQuality: An Open-Source Software Tool for Quality Monitoring and Reporting
      of Targeted Mass Spectrometry Measurements'
  source_content_hash: sha256:6ec814e0382b541c77e95d516ca905c5db13374da351d2ba790bb428bd4ce6e7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/hankemeierlab/mzQuality
  tools:
  - R
  - R-Shiny
  - mzQuality
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5384615384615384
n_claims: 13
---

# lc-ms-qc-sample-pca-visualization

## Summary

Generates PCA plots comparing study samples and pooled study quality control (SQC) samples across batches in mzQuality, before and after between-batch correction, to judge whether batch effects on a targeted LC-MS metabolomics dataset have been successfully removed.

## When to use

Use when a targeted LC-MS metabolomics study was measured in multiple batches and you need to verify, via PCA of sample types and batches, whether pooled QC (SQC) samples become aligned and centered across batches after batch correction.

## When NOT to use

- For untargeted metabolomics peak tables processed with tools like OpenMS or MZmine rather than mzQuality's SQC-based SummarizedExperiment workflow — this skill depends on mzQuality's batch-corrected 'ratio_corrected' assay and sample-type annotations.
- When you only need per-compound QC relative standard deviation (RSD) checks rather than a multivariate view of between-batch alignment; the RSDqc <= 30% threshold tabulation is a separate evaluation.
- When samples are not measured in multiple batches with pooled QC (SQC) samples, since the PCA's purpose here is comparing SQC vs study samples across batches.

## Inputs

- Peak-area table from targeted LC-MS (e.g. tab-delimited file with mandatory columns as checked by readData, or a SummarizedExperiment)
- SummarizedExperiment object built by mzQuality's buildExperiment
- Batch-corrected compound/IS ratio assay (e.g. assay 'ratio_corrected') produced by doAnalysis

## Outputs

- PCA plot showing the aliquot distribution per sample type (study samples vs SQC) and batch for batch-corrected ratios
- Per-batch confidence interval (CI) ellipses in the PCA plot for judging alignment of SQC samples
- Complementary violin plots showing the QC sample distribution per batch before and after correction

## How to apply

Load the packaged example peak-area table with `system.file("extdata", "example.tsv", package = "mzQuality")`, read it with `readData` (which checks for mandatory columns), and build the experiment with `buildExperiment`, producing a SummarizedExperiment. Run `doAnalysis(exp)`, which computes compound/IS ratios, applies between-batch correction using the pooled study quality control samples (SQC), and adds assays such as `ratio` and `ratio_corrected`; flagged potential outliers are automatically excluded from calculations. Call `pcaPlot(exp, assay = "ratio_corrected")` to generate the post-correction PCA showing aliquot distribution per type and batch; an uncorrected view can be made with the `ratio` assay. Accept the correction if the post-correction PCA shows SQC samples aligned and mostly centered within the study samples with overlapping confidence interval (CI) ellipses across batches — in the article's 419-sample, six-batch example, the two CI ellipses overlapped after correction, whereas before correction samples and SQC samples from the two batches were not aligned. Violin plots (`violinPlot`) of QC sample distribution per batch can complement the PCA view of variation.

## Related tools

- **mzQuality** (R package whose pcaPlot function generates the PCA plot of aliquot distribution per type and batch for batch-corrected ratios; its doAnalysis produces the batch-corrected assay used as PCA input.) — https://github.com/hankemeierlab/mzQuality
- **R** (Runtime environment for the mzQuality package (R 4.0 or later required) in which pcaPlot and violinPlot are executed.)
- **R-Shiny** (Framework behind the mzQualityDashboard interface, recommended for interactive use of these QC visualizations by new R users.)

## Examples

```
path <- system.file("extdata", "example.tsv", package = "mzQuality"); exp <- buildExperiment(readData(path)); exp <- doAnalysis(exp); pcaPlot(exp, assay = "ratio_corrected")
```

## Evaluation signals

- After correction, the PCA plot shows SQC samples aligned and mostly centered within the study samples, with overlapping CI ellipses across the six batches (matching the article's six-batch, 419-sample example).
- The uncorrected 'ratio' assay PCA contrasts with the corrected one: samples and SQC samples from the two batches were not aligned before correction.
- pcaPlot is called with assay = 'ratio_corrected' (batch-corrected ratios), not the raw 'ratio' assay, for the 'after' diagnostic.
- Sample-type grouping in the PCA distinguishes study samples from pooled study quality control (SQC) samples, with batches mapped to colors/ellipses as in aliquot distribution per type and batch.
- Complementary QC violin plots show reduced per-batch spread after correction, consistent with the PCA-based conclusion.

## Limitations

- Missing values (NA) are treated as absent data points and samples with NA will not have a data point shown, which can thin the PCA representation for affected samples.
- Compounds not detected in LQC or SQC samples are assumed not present and are not reported, so such compounds are excluded from the corrected ratios feeding the PCA.
- Potential outliers flagged by mzQuality (e.g. via the Rosner Test on QC samples) are automatically excluded from calculations, so the PCA reflects only retained samples unless the 'use' flags are overridden manually.
- PCA here is a QC/batch-alignment diagnostic on targeted compound ratios, not a hypothesis-testing multivariate model.

## Evidence

- [readme] PCA plot purpose and assay choice for the after-correction view: [evidence span withheld — anchor status: not_found]
- [results] Article statement of the before/after PCA outcome that validates the correction: [evidence span withheld — anchor status: not_found]
- [readme] doAnalysis performs batch correction using pooled study QC samples: [evidence span withheld — anchor status: external_doc]
- [abstract] mzQuality visualizes data via PCA plots among other QC views: "mzQuality visualizes data through sample plots, PCA plots, and violin plots"
- [results] Potential outliers are excluded from calculations feeding the corrected data: "Potential outliers are flagged by mzQuality and are automatically excluded from further calculations"
