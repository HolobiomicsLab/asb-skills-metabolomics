---
name: shiny-app-lipidomics-workflow-execution
description: Use when lipidomics data must be imported from the Metabolomics Workbench
  (e.g. a healthy-vs-diabetic serum experiment) or from LipidSearch/LIQUID outputs,
  then parsed, normalized, and tested for differential lipid abundance.
when_to_use_negative:
- The input is already a preprocessed/normalized feature table and the goal is only
  bespoke statistics outside the app's LipidSearch/LIQUID/Metabolomics Workbench parsing
  scope
- No internet connection is available and the Metabolomics Workbench import is required,
  since do_query()-backed download needs online access
- The instrument/vendor output is neither LipidSearch, LIQUID, nor a Metabolomics
  Workbench experiment
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: ADViSELipidomics
  role: Shiny app executing the full preprocessing, analysis, and visualization workflow
    on lipidomics data
  repo: https://github.com/ShinyFabio/ADViSELipidomics
- name: metabolomicsWorkbenchR
  role: Provides the do_query() function used to download Metabolomics Workbench lipidomic
    experiments in real time
- name: Metabolomics Workbench
  role: Online repository from which suitable selected lipidomic experiments (e.g.
    the healthy-vs-diabetic serum study) are retrieved
- name: Shiny
  role: Framework in which the ADViSELipidomics application is implemented and launched
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btac706
    title: 'ADViSELipidomics: a workflow for analyzing lipidomics data'
  source_content_hash: sha256:97e3bb25eee3b87f4ea514a3145cc305fdf3b4f398c1c8944100f63375ffac81
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ShinyFabio/ADViSELipidomics
  tools:
  - ADViSELipidomics
  - Metabolomics Workbench
  - Shiny
  - metabolomicsWorkbenchR
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 15
---

# shiny-app-lipidomics-workflow-execution

## Summary

Executes the ADViSELipidomics Shiny workflow for preprocessing, analyzing, and visualizing lipidomics data, including real-time import of selected lipidomic experiments from the Metabolomics Workbench repository for differential analysis tasks such as healthy-vs-diabetic serum lipid profiles.

## When to use

Use when lipidomics data must be imported from the Metabolomics Workbench (e.g. a healthy-vs-diabetic serum experiment) or from LipidSearch/LIQUID outputs, then parsed, normalized, and tested for differential lipid abundance.

## When NOT to use

- The input is already a preprocessed/normalized feature table and the goal is only bespoke statistics outside the app's LipidSearch/LIQUID/Metabolomics Workbench parsing scope
- No internet connection is available and the Metabolomics Workbench import is required, since do_query()-backed download needs online access
- The instrument/vendor output is neither LipidSearch, LIQUID, nor a Metabolomics Workbench experiment

## Inputs

- Metabolomics Workbench lipidomic experiment (selected/downloadable in real time via metabolomicsWorkbenchR do_query())
- LipidSearch output files for lipid identification and quantification
- LIQUID output files for lipid identification and quantification
- Sample metadata associated with the experiment (for the healthy-vs-diabetic design)
- Internal lipid standards present in the experiment (required for normalization)

## Outputs

- Parsed lipid species table using LIPID MAPS classification
- Normalized concentration values per lipid and per sample (absolute values when internal standards are present)
- List of differentially abundant lipids with statistics for the chosen experimental design
- Exploratory and statistical visualizations of the lipidomics data

## How to apply

Launch ADViSELipidomics either from R (> 4.0) via devtools::install_github("ShinyFabio/ADViSELipidomics") followed by run_ADViSELipidomics(), or via Docker with docker run -p 80:80 shinyfabio/adviselipidomics:latest and open http://localhost:80/. Use the built-in Metabolomics Workbench import, backed by the do_query() function from the {metabolomicsWorkbenchR} package, to download the selected lipidomic experiment in real time (internet connection required). Preprocess by parsing lipid species using the LIPID MAPS classification together with sample metadata; when the experiment includes internal lipid standards, normalize the data matrix to obtain normalized concentration values per lipid and per sample. Run differential abundance analysis for the simple (e.g. healthy-vs-diabetic) or complex experimental design, applying batch effect correction if needed. Validate by comparing the resulting differential lipid list against the corresponding case study results reported in the paper as a qualitative acceptance check (no quantitative threshold is stated).

## Related tools

- **ADViSELipidomics** (Shiny app executing the full preprocessing, analysis, and visualization workflow on lipidomics data) — https://github.com/ShinyFabio/ADViSELipidomics
- **metabolomicsWorkbenchR** (Provides the do_query() function used to download Metabolomics Workbench lipidomic experiments in real time)
- **Metabolomics Workbench** (Online repository from which suitable selected lipidomic experiments (e.g. the healthy-vs-diabetic serum study) are retrieved)
- **Shiny** (Framework in which the ADViSELipidomics application is implemented and launched)

## Examples

```
devtools::install_github("ShinyFabio/ADViSELipidomics"); library("ADViSELipidomics"); run_ADViSELipidomics()
```

## Evaluation signals

- The Metabolomics Workbench experiment downloads successfully in real time via do_query() with an active internet connection
- Lipid species are parsed according to the LIPID MAPS classification and joined with sample metadata
- Normalized concentration values per lipid and per sample are produced when internal lipid standards are present in the experiment
- The differential abundance analysis returns a differentially abundant lipid list for the simple healthy-vs-diabetic design, with batch effect correction applied if needed
- The differential lipid list and statistics qualitatively match the Case Study 2 results reported in the paper (no quantitative threshold is reported in the text)

## Limitations

- The interface layout can differ depending on screen size and monitor resolution, requiring zoom adjustments (Ctrl +/- or Command +/-)
- The Metabolomics Workbench import only covers 'suitable selected lipidomic experiments' from the repository and requires an internet connection
- The specific batch correction method used is not identified in the paper (the method name is not given)
- Power to detect non-linear drift in QC samples is low: the 2D pooled distribution pattern is not visible in replicate (n=1 vs n=2) QC samples because variance is very low
- Installation may require OS-specific dependencies (Rtools on Windows; Homebrew imagemagick@6 and cairo on macOS; several apt packages on Ubuntu) and compilation can take a long time; Firefox browser is noted to have some problems with the Docker version

## Evidence

- [supplementary] ADViSELipidomics can download in real-time suitable selected lipidomic experiments from the online repository: "a Metabolomics Workbench lipidomic experiment. ADViSELipidomics can download in real-time suitable selected lipidomic experiments from the online repository"
- [supplementary] do_query from metabolomicsWorkbenchR: "using the `do_query()` function from the `{metabolomicsWorkbenchR}` package"
- [abstract] Normalization with internal standards: "When the experiment includes internal lipid standards, ADViSELipidomics can normalize the data matrix, providing normalized concentration values per lipids and samples."
- [abstract] Differential abundance and batch correction: "it identifies differentially abundant lipids in simple and complex experimental designs, dealing with batch effect correction."
- [methods] Launch commands: "library("ADViSELipidomics")
run_ADViSELipidomics()"
- [methods] Docker launch: "docker run -p 80:80 shinyfabio/adviselipidomics:latest"
- [readme] Readme internet requirement: "Internet
connection is required."
