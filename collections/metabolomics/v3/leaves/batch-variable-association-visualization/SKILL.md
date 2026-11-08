---
name: batch-variable-association-visualization
description: Use when a cohort metabolomics dataset (e.g. Metabolon OrigScale sheet
  input) has passed the metaboprep QC pipeline and you must visualise/quantify how
  available batch variables influence the data in a disclosure-ready HTML report.
when_to_use_negative:
- The Metaboprep object has not yet passed quality_control — summaries, PC plots and
  the dendrogram will not be populated, so the qc_report sections (including batch-variable
  panels) cannot render correctly.
- You need a standalone statistical batch-effect test on raw data rather than the
  package's standardized disclosure report; metaboprep provides a report of influence
  of available batch variables, not a custom modelling workflow.
- The input is not metabolomics/omics feature data loadable into a Metaboprep object
  (e.g. an already filtered feature table without sample batch annotations).
edam_operation: http://edamontology.org/operation_0306
edam_topics:
- http://edamontology.org/topic_2269
- http://edamontology.org/topic_3520
tools:
- name: metaboprep
  role: Provides the qc_report rmarkdown template rendered by generate_report, containing
    the batch-variable association boxplots with type-II ANOVA eta-squared annotations
  repo: https://github.com/MRCIEU/metaboprep
- name: rmarkdown
  role: Engine that renders the metaboprep qc_report template into the HTML report
- name: R
  role: Environment in which the metaboprep QC pipeline and report generation are
    run
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioinformatics/btac059
    title: 'metaboprep: an R package for preanalysis data description and processing'
  source_content_hash: sha256:6c96573b299ee0b863ee6f654c447e676f66036951f9758d0d682f7765390add
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/MRCIEU/metaboprep
  tools:
  - R
  - metaboprep
  - rmarkdown
  edam_operation: http://edamontology.org/operation_0306
  edam_topics:
  - http://edamontology.org/topic_2269
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# batch-variable-association-visualization

## Summary

Generates the metaboprep qc_report HTML report whose batch-variable section plots associations between available batch variables (e.g. run_day, box_id, lot) and the metabolomics data as boxplots annotated with type-II ANOVA eta-squared estimates, for open disclosure of preanalysis data quality.

## When to use

Use when a cohort metabolomics dataset (e.g. Metabolon OrigScale sheet input) has passed the metaboprep QC pipeline and you must visualise/quantify how available batch variables influence the data in a disclosure-ready HTML report.

## When NOT to use

- The Metaboprep object has not yet passed quality_control — summaries, PC plots and the dendrogram will not be populated, so the qc_report sections (including batch-variable panels) cannot render correctly.
- You need a standalone statistical batch-effect test on raw data rather than the package's standardized disclosure report; metaboprep provides a report of influence of available batch variables, not a custom modelling workflow.
- The input is not metabolomics/omics feature data loadable into a Metaboprep object (e.g. an already filtered feature table without sample batch annotations).

## Inputs

- Metaboprep S7 object that has completed the quality_control pipeline (input and qc data layers, sample/feature summaries, PC and dendrogram content populated)
- Cohort metabolomics dataset, e.g. Metabolon-formatted xlsx read via read_metabolon with sheet='OrigScale'
- Sample metadata batch variables (e.g. run_day, box_id, lot) carried in the Metaboprep object's sample annotation
- Project name and output directory for the rendered report

## Outputs

- {project}_metaboprep_qc_report.html (e.g. BiB_MS-1_metaboprep_qc_report.html, ALSPAC_F24_metaboprep_qc_report.html)
- Batch-variable association boxplots annotated with type-II ANOVA eta-squared estimates
- Accompanying report panels: raw-data overview, exclusion summary, filtered-dataset summary, Shapiro W-statistic histograms, PC plots, feature-clustering dendrogram
- Tab-delimited summary text files emitted alongside the HTML report

## How to apply

First run the metaboprep quality_control pipeline on the Metaboprep S7 object with the reported thresholds (sample_missingness=0.2, feature_missingness=0.2, total_peak_area_sd=5, outlier_udist=5, outlier_treatment='leave_be', winsorize_quantile=1.0, tree_cut_height=0.5, pc_outlier_sd=5) so sample/feature summaries, PC plots and the feature dendrogram are populated — the batch-variable visualisation is part of the qc_report template, not a standalone call. Then call generate_report(metaboprep, project=<cohort name>, output_dir, format='html', template='qc_report') once per cohort (e.g. 'BiB_MS-1' and 'ALSPAC_F24') to render {project}_metaboprep_qc_report.html. In the rendered HTML, confirm the batch-variable association boxplots are annotated with type-II ANOVA eta-squared estimates, alongside the raw-data overview, exclusion summary, filtered-dataset summary, Shapiro W-statistic histograms, PC plots and feature-clustering dendrogram, matching the deposited Supplementary Data S2 and S3 reports.

## Related tools

- **metaboprep** (Provides the qc_report rmarkdown template rendered by generate_report, containing the batch-variable association boxplots with type-II ANOVA eta-squared annotations) — https://github.com/MRCIEU/metaboprep
- **rmarkdown** (Engine that renders the metaboprep qc_report template into the HTML report)
- **R** (Environment in which the metaboprep QC pipeline and report generation are run)

## Examples

```
generate_report(metaboprep = mydata, project = 'ALSPAC_F24', output_dir = outdir, format = 'html', template = 'qc_report')
```

## Evaluation signals

- The output directory contains the rendered file named {project}_metaboprep_qc_report.html after each generate_report call.
- The HTML report displays batch-variable association boxplots annotated with type-II ANOVA eta-squared estimates.
- The report also shows the raw-data overview, exclusion summary, filtered-dataset summary, Shapiro W-statistic histograms, PC plots and the feature-clustering dendrogram, matching the deposited Supplementary Data S2 and S3.
- The exclusion summary counts reflect the QC thresholds applied (e.g. user_defined_sample_missingness exclusions at the 0.2 threshold), confirming the pipeline ran before reporting.
- Rendering succeeded per cohort (BiB_MS-1 and ALSPAC_F24) with report content consistent across the two cohorts' structure.

## Limitations

- The report's visual content depends on the QC pipeline having been executed with the stated thresholds; an unprocessed object yields empty summaries and plots.
- The package is marked experimental in its README lifecycle badge, so report APIs may change.
- PCA-based reporting can warn when the stated max_num_pcs exceeds the number of available informative PCs (as seen in the README example: max_num_pcs=10 vs 2 informative PCs), which can affect PC-plot content.

## Evidence

- [abstract] Report summarizes quality metrics and batch-variable influence for disclosure: "A report summarizing quality metrics and the influence of available batch variables on the data are generated for the purpose of open disclosure"
- [intro] Package outputs include an HTML report: "Provide useful summary data in the form of tab-delimited text file and a html report."
- [readme] README describes goal of producing HTML summary reports: "Provide useful summary data in the form of tab-delimited text file and a html report."
