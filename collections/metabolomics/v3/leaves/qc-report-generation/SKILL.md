---
name: qc-report-generation
description: Use when a metabolomics QC pipeline (e.g. metaboprep quality_control
  on a Metaboprep object) has finished and you need a rendered HTML report documenting
  sample/feature quality metrics and batch effects for disclosure.
when_to_use_negative:
- Input is raw vendor output that has not yet been loaded into a Metaboprep object
  and passed through quality_control — build the object first with read_metabolon/Metaboprep
- The analysis goal is data filtering itself (threshold selection on sample_missingness,
  feature_missingness, total_peak_area_sd, etc.) rather than documentation of results
- A non-metaboprep QC framework (e.g. an independent Python pipeline) is in use; the
  report templates are tied to the metaboprep Metaboprep object structure
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: metaboprep
  role: Provides the Metaboprep object, quality_control, feature_summary/sample_summary,
    and report generation used to build the QC report
  repo: https://github.com/MRCIEU/metaboprep
- name: R
  role: Runtime environment in which the metaboprep reporting workflow is executed
  repo: https://github.com/MRCIEU/metaboprep
- name: readxl
  role: Reads the metabolon_v1.1_example.xlsx input worksheet (sheet = 'OrigScale')
    consumed by read_metabolon prior to reporting
- name: rmarkdown
  role: Rendering backend for the HTML QC report output
- name: knitr
  role: Chunk engine used in the report vignette rendering
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/bioinformatics/btac059
    title: 'metaboprep: an R package for preanalysis data description and processing'
  source_content_hash: sha256:5ed2315bd3582c4c0035c1ed7657f1f8cb2c2e91e228d5b7d3eea23d9fbbd8fd
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/MRCIEU/metaboprep
  tools:
  - R
  - knitr
  - metaboprep
  - readxl
  - rmarkdown
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# qc-report-generation

## Summary

Generate an open-disclosure report from a metaboprep Metaboprep object that summarizes quality metrics, batch-variable influence, and per-metabolite diagnostics (scatter plots of outlying points, value histograms, and summary-statistic tables).

## When to use

Use when a metabolomics QC pipeline (e.g. metaboprep quality_control on a Metaboprep object) has finished and you need a rendered HTML report documenting sample/feature quality metrics and batch effects for disclosure.

## When NOT to use

- Input is raw vendor output that has not yet been loaded into a Metaboprep object and passed through quality_control — build the object first with read_metabolon/Metaboprep
- The analysis goal is data filtering itself (threshold selection on sample_missingness, feature_missingness, total_peak_area_sd, etc.) rather than documentation of results
- A non-metaboprep QC framework (e.g. an independent Python pipeline) is in use; the report templates are tied to the metaboprep Metaboprep object structure

## Inputs

- Metaboprep object with completed quality_control on the input data layer (input and qc layers with sample/feature summaries)
- Project name, output directory, report format ('html'), and report template (e.g. 'qc_report')

## Outputs

- Rendered HTML QC report (e.g. myproject_metaboprep_qc_report.html) summarizing quality metrics and batch-variable influence
- Per-metabolite scatter plot of outlying data points
- Per-metabolite histogram of values
- Per-metabolite table of selected summary statistics
- Companion tab-delimited summary text files (feature_summary / sample_summary outputs)

## How to apply

After running quality_control on the input layer of a Metaboprep object (built via read_metabolon with sheet = 'OrigScale', return_Metaboprep = TRUE from a metabolon_v1.1_example.xlsx-style file), call generate_report with project, output_dir = getwd(), format = 'html', and template = 'qc_report' to render the report. The report condenses the summary statistics computed by feature_summary and sample_summary and the influence of available batch variables (e.g. run_day, box_id, lot in the metabolon example) on the data. Per metabolite it emits a scatter plot flagging outlying data points, a histogram of the values, and a table of selected summary statistics. Rationale: the article states reports are 'generated for the purpose of open disclosure' to address the lack of standardization and reporting transparency in preanalytical metabolomics processing. Validate by checking the output file (e.g. myproject_metaboprep_qc_report.html) exists and renders as an HTML document.

## Related tools

- **metaboprep** (Provides the Metaboprep object, quality_control, feature_summary/sample_summary, and report generation used to build the QC report) — https://github.com/MRCIEU/metaboprep
- **R** (Runtime environment in which the metaboprep reporting workflow is executed) — https://github.com/MRCIEU/metaboprep
- **readxl** (Reads the metabolon_v1.1_example.xlsx input worksheet (sheet = 'OrigScale') consumed by read_metabolon prior to reporting)
- **rmarkdown** (Rendering backend for the HTML QC report output)
- **knitr** (Chunk engine used in the report vignette rendering)

## Examples

```
mydata |> generate_report(project = "myproject", output_dir = getwd(), format = "html", template = "qc_report")
```

## Evaluation signals

- Output file exists in output_dir with the expected name (e.g. myproject_metaboprep_qc_report.html) and is a rendered HTML document
- Report contains per-metabolite scatter plot, histogram, and summary-statistic table for each metabolite in the dataset
- Report summary tables reproduce the feature_summary/sample_summary statistics from the Metaboprep object (e.g. the 100-sample, 100-feature example object with input and qc layers)
- Report reflects the QC exclusions applied (e.g. 2 samples excluded via user_defined_sample_missingness at >=20%)
- Batch-variable influence sections correspond to annotation columns present in the Metaboprep object (e.g. run_day, box_id, lot)

## Limitations

- The report documents the metaboloprep QC pipeline; it does not itself perform filtering or threshold selection
- Report content depends on which batch variables are available in sample annotation; with no batch metadata, influence sections are empty
- metaboprep is marked as experimental (lifecycle badge), so report APIs/formats may change between development versions (e.g. the v2_development branch)

## Evidence

- [abstract] Report summarizing quality metrics and batch influence: "A report summarizing quality metrics and the influence of available batch variables on the data are generated for the purpose of open disclosure"
- [intro] Summary data outputs: "Provide useful summary data in the form of tab-delimited text file and a html report."
- [abstract] Standardized processing workflow: "we introduce metaboprep, a standardized data processing workflow to extract and characterize high quality metabolomics datasets"
- [readme] Goal includes html report: "2.  Provide useful summary data in the form of tab-delimited text file and a html report."
- [readme] QC pipeline produces the summaries that feed the report: [evidence span withheld — anchor status: external_doc]
