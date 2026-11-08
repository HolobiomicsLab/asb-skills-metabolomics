---
name: random-forest-classification
description: Use when metabolomics or proteomics feature tables (CSV features + target/metadata
  files, e.g. ST000284 or ST000336) need exploratory visualization and quality checks
  before statistical analysis.
when_to_use_negative:
- Input is raw spectral data requiring MS processing — the article directs such users
  to the RforMassSpectrometry packages first and then migrate to POMA
- The goal is statistical hypothesis testing or machine-learning prediction (POMAShiny
  panels 3–4); EDA is a preceding exploratory step, not a substitute
- Data cannot be supplied as POMAShiny's two required CSV files (target metadata +
  features)
edam_operation: http://edamontology.org/operation_0337
tools:
- name: POMAShiny
  role: Web application whose EDA panel performs the exploratory data analysis as
    panel 3 of its four-panel workflow
  repo: https://github.com/nutrimetabolomics/POMAShiny
- name: POMA
  role: Underlying R/Bioconductor package that backs POMAShiny, allowing the same
    exploratory analysis to be reproduced outside the web environment
  repo: https://github.com/nutrimetabolomics/POMA
- name: Shiny
  role: Framework powering the POMAShiny interface through which the EDA panel is
    run (shiny::runApp)
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1371/journal.pcbi.1009148
    title: 'POMAShiny: A user-friendly web-based workflow for metabolomics and proteomics
      data analysis'
  source_content_hash: sha256:06694caf757a2a67fcf44d57aedc2f751ade53e85f60e36e14a7ba308f8e3047
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/nutrimetabolomics/POMAShiny
  tools:
  - POMA
  - POMAShiny
  - R
  - Shiny
  edam_operation: http://edamontology.org/operation_0337
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8461538461538461
n_claims: 13
---

# Exploratory data analysis (EDA)

## Summary

The exploratory data analysis (EDA) panel of POMAShiny, the third step in its four-panel workflow (data upload, preprocessing, EDA, statistical analysis), which the authors recommend performing before any statistical analysis of metabolomics or proteomics data.

## When to use

Use when metabolomics or proteomics feature tables (CSV features + target/metadata files, e.g. ST000284 or ST000336) need exploratory visualization and quality checks before statistical analysis.

## When NOT to use

- Input is raw spectral data requiring MS processing — the article directs such users to the RforMassSpectrometry packages first and then migrate to POMA
- The goal is statistical hypothesis testing or machine-learning prediction (POMAShiny panels 3–4); EDA is a preceding exploratory step, not a substitute
- Data cannot be supplied as POMAShiny's two required CSV files (target metadata + features)

## Inputs

- Target (metadata) CSV file
- Features CSV file (sample × feature intensity matrix)
- Example datasets ST000284 (three-group targeted metabolomics) and ST000336 (two-group) from Metabolomics Workbench

## Outputs

- Interactive exploratory visualizations of the preprocessed feature matrix within the POMAShiny web interface

## How to apply

Provide POMAShiny's two required inputs: a target (metadata) CSV file and a features CSV file, loaded through the data upload panel. Optionally pass the data through the preprocessing panel first: remove features with more than 20% missing values per study group (default threshold), impute missing values using one of six methods (zero, half minimum, median, mean, minimum, k-nearest neighbours), normalize via transformation and scaling (autoscaling, level scaling, log scaling, log transformation, vast scaling, log pareto scaling), and remove statistical outliers with customizable parameters. Then run the EDA panel — the article states it is highly recommended to perform EDA before any statistical analysis. Deploy the app locally with shiny::runApp(appDir = 'app') or via the Docker image pcastellanoescuder/pomashiny:v.1.1.14, and ground the exploration on the bundled example datasets ST000284 (targeted metabolomics three-group study) and ST000336 (two-group study) from Metabolomics Workbench.

## Related tools

- **POMAShiny** (Web application whose EDA panel performs the exploratory data analysis as panel 3 of its four-panel workflow) — https://github.com/nutrimetabolomics/POMAShiny
- **POMA** (Underlying R/Bioconductor package that backs POMAShiny, allowing the same exploratory analysis to be reproduced outside the web environment) — https://github.com/nutrimetabolomics/POMA
- **Shiny** (Framework powering the POMAShiny interface through which the EDA panel is run (shiny::runApp))

## Examples

```
shiny::runApp(appDir = "app")
```

## Evaluation signals

- Both required input files (target metadata CSV and features CSV) are present before the EDA panel is invoked
- Preprocessing decisions are documented: missing-value feature removal threshold (default 20% per study group), chosen imputation method (of six), and chosen normalization method (of six)
- EDA is executed before the statistical analysis panel, per the article's recommendation
- The application launches successfully via shiny::runApp(appDir = "app") or docker run -d --rm -p 3838:3838 pomashiny
- Results are consistent with the example ST000284 three-group or ST000336 two-group structure when those datasets are used as a sanity check

## Limitations

- POMAShiny was built with bs4Dash version 0.5.0 and is not compatible with newer versions due to breaking API changes (README)
- No benchmarking or quantitative evaluation of POMAShiny against other tools is reported
- No changelog or explicit software version number is stated for POMAShiny or POMA
- Requires the two-CSV input format; data in other layouts must be converted first

## Evidence

- [results] EDA recommended before statistics: "it is highly recommended to perform an EDA before any statistical analysis"
- [results] Two CSV inputs: "POMAShiny requires two comma-separated value (CSV) files as an input: the target (or metadata) file and the features file"
- [intro] Workflow coverage: "POMAShiny provides a comprehensive and structured workflow that covers the preprocessing, exploratory data analysis and statistical analysis of these data"
- [results] Four sequential panels: "1) data upload, 2) preprocessing, 3) exploratory data analysis (EDA) and 4) statistical analysis"
- [readme] README panel structure: "POMAShiny’s workflow is structured in four sequential and well-defined panels"
