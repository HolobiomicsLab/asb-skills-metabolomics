---
name: r-html-report-rendering
description: Use when a metaboprep object has completed the quality_control pipeline
  and you must produce an HTML disclosure report summarizing quality metrics, filtering
  steps, and batch-variable influence for a cohort.
when_to_use_negative:
- The metaboprep object has not yet gone through quality_control — summaries, PC plots,
  and the dendrogram will be unpopulated, so the report would be empty or fail to
  render
- You only need machine-readable summary statistics — the package also emits tab-delimited
  text summaries, which are the appropriate output rather than the HTML report
- The data is not loaded into a Metaboprep object (e.g., a raw feature table or non-metabolomics
  omics data outside the package's read_metabolon/readers scope)
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3489
tools:
- name: metaboprep
  role: Provides the Metaboprep object, the quality_control pipeline whose qc layer
    feeds the report, and the report generation that summarizes quality metrics and
    batch-variable influence
  repo: https://github.com/MRCIEU/metaboprep
- name: rmarkdown
  role: Rendering engine for the qc_report HTML vignette/template that produces the
    disclosed HTML report
- name: R
  role: Runtime environment in which the metaboprep object is constructed, QC is run,
    and the HTML report is rendered
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioinformatics/btac059
    title: 'metaboprep: an R package for preanalysis data description and processing'
  source_content_hash: sha256:18e46b30b5fae778fb7388664271b54ab95041b3b1e26685d6dcb2ac533dca89
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/MRCIEU/metaboprep
  tools:
  - R
  - metaboprep
  - rmarkdown
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3489
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 5
---

# r-html-report-rendering

## Summary

Render a standardized self-contained HTML quality-control report from a completed metaboprep S7 object via the package's qc_report rmarkdown template, so that quality metrics, filtering decisions, and batch-variable effects are disclosed in a reproducible document (e.g., deposited as Supplementary Data S2/S3 for BiB_MS-1 and ALSPAC_F24).

## When to use

Use when a metaboprep object has completed the quality_control pipeline and you must produce an HTML disclosure report summarizing quality metrics, filtering steps, and batch-variable influence for a cohort.

## When NOT to use

- The metaboprep object has not yet gone through quality_control — summaries, PC plots, and the dendrogram will be unpopulated, so the report would be empty or fail to render
- You only need machine-readable summary statistics — the package also emits tab-delimited text summaries, which are the appropriate output rather than the HTML report
- The data is not loaded into a Metaboprep object (e.g., a raw feature table or non-metabolomics omics data outside the package's read_metabolon/readers scope)

## Inputs

- Metaboprep S7 object with a populated 'qc' data layer (post quality_control with thresholds sample_missingness=0.2, feature_missingness=0.2, total_peak_area_sd=5, outlier_udist=5, tree_cut_height=0.5, pc_outlier_sd=5)
- Cohort metabolomics dataset previously read via read_metabolon (e.g., Metabolon-formatted xlsx, sheet='OrigScale') for cohorts such as BiB_MS-1 and ALSPAC_F24
- Project name string (e.g., 'BiB_MS-1', 'ALSPAC_F24') and an output directory path

## Outputs

- <project>_metaboprep_qc_report.html rendered from the qc_report rmarkdown template (e.g., BiB_MS-1_metaboprep_qc_report.html, ALSPAC_F24_metaboprep_qc_report.html)
- Report sections: raw-data overview, exclusion summary, filtered-dataset summary, Shapiro W-statistic histograms, PC plots, feature-clustering dendrogram, batch-variable association boxplots with type-II ANOVA eta-squared estimates

## How to apply

First ensure the Metaboprep object has been processed by quality_control with the study's thresholds (sample_missingness=0.2, feature_missingness=0.2, total_peak_area_sd=5, outlier_udist=5, outlier_treatment='leave_be', winsorize_quantile=1.0, tree_cut_height=0.5, pc_outlier_sd=5), because the report's summaries, PC plots, and feature-clustering dendrogram are populated from the qc layer of that object. Then invoke the report-generation step per project (e.g., project='BiB_MS-1' and project='ALSPAC_F24'), rendering the 'qc_report' rmarkdown template to <project>_metaboprep_qc_report.html in a specified output directory. The rendered document must display the raw-data overview, exclusion summary, filtered-dataset summary, Shapiro W-statistic histograms, PC plots, the feature-tree dendrogram, and batch-variable association boxplots annotated with type-II ANOVA eta-squared estimates. The rationale is open disclosure: the package's stated goal is to summarize quality metrics and the influence of available batch variables so preanalytical processing is transparent. Verify by checking the output directory contains the HTML file and that each expected section/plot matches the deposited supplementary report.

## Related tools

- **metaboprep** (Provides the Metaboprep object, the quality_control pipeline whose qc layer feeds the report, and the report generation that summarizes quality metrics and batch-variable influence) — https://github.com/MRCIEU/metaboprep
- **rmarkdown** (Rendering engine for the qc_report HTML vignette/template that produces the disclosed HTML report)
- **R** (Runtime environment in which the metaboprep object is constructed, QC is run, and the HTML report is rendered)

## Examples

```
mydata <- mydata |> quality_control(source_layer="input", sample_missingness=0.2, feature_missingness=0.2, total_peak_area_sd=5, outlier_udist=5, outlier_treatment="leave_be", winsorize_quantile=1.0, tree_cut_height=0.5, pc_outlier_sd=5)
```

## Evaluation signals

- Output directory contains the rendered file named <project>_metaboprep_qc_report.html for each project (BiB_MS-1 and ALSPAC_F24)
- Report displays all expected sections: raw-data overview, exclusion summary, filtered-dataset summary, Shapiro W-statistic histograms, PC plots, and the feature-clustering dendrogram
- Batch-variable association boxplots in the report carry type-II ANOVA eta-squared estimates, matching the deposited Supplementary Data S2/S3 content
- Exclusion counts in the report's summary (e.g., user_defined_sample_missingness exclusions) agree with the Metaboprep object's Exclusion Codes Summary after quality_control
- Report content only appears if the qc layer was populated — an object that skipped quality_control yields missing/empty summaries, PC plots, and dendrogram

## Limitations

- The package carries an 'experimental' lifecycle badge, so report-rendering behavior may change between development versions
- The report is only as complete as the underlying qc layer: PCA outlier assessment can warn that the stated max_num_pcs (e.g., 10) exceeds the number of available informative PCs, affecting the PC plots the report can show
- Report content is tied to metaboprep's qc_report template; it covers quality metrics and batch-variable influence but is not a general-purpose analysis report

## Evidence

- [abstract] A report summarizing quality metrics and the influence of available batch variables on the data are generated for the purpose of open disclosure: "A report summarizing quality metrics and the influence of available batch variables on the data are generated for the purpose of open disclosure"
- [intro] Provide useful summary data in the form of tab-delimited text file and a html report.: "Provide useful summary data in the form of tab-delimited text file and a html report."
- [methods] output: rmarkdown::html_vignette: "output: rmarkdown::html_vignette"
- [readme] The goal of `metaboprep` is to: 1. Read in and processes (un)targeted metabolite data: "The goal of `metaboprep` is to:

1.  Read in and processes (un)targeted metabolite data, saving datasets
    in tab-delimited format for use elsewhere
2.  Provide useful summary data in the form of"
