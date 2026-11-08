---
name: lipidomics-dimensionality-reduction
description: Use when a preprocessed lipidomics data matrix (from LipidSearch/LIQUID
  outputs or a Metabolomics Workbench experiment) is loaded in the ADViSELipidomics
  Shiny app and you need unsupervised/supervised dimension reduction, sample clustering,
  or interactive exploratory visualizations.
when_to_use_negative:
- The goal is differential abundance testing, internal-standard-based normalization
  to absolute concentrations, or batch effect correction — these are separate ADViSELipidomics
  capabilities, not the exploratory dimension-reduction stage
- The input is raw LipidSearch/LIQUID identification output that has not yet been
  preprocessed into a lipid-by-sample abundance matrix
tools:
- name: ADViSELipidomics
  role: R/Shiny application whose exploratory-analysis layer performs PCA, PLS-DA
    and clustering on the preprocessed data and renders the interactive plots
  repo: https://github.com/ShinyFabio/ADViSELipidomics
- name: Shiny
  role: Framework underlying the ADViSELipidomics app that provides the interactive,
    customizable plot outputs
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btac706
    title: 'ADViSELipidomics: a workflow for analyzing lipidomics data'
  source_content_hash: sha256:08bce3142fcc2f3c444911b690a82f72ade166b51ad501fc723dd990ae387cdd
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ShinyFabio/ADViSELipidomics
  tools:
  - ADViSELipidomics
  - R
  - Shiny
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# lipidomics-dimensionality-reduction

## Summary

Applies the exploratory-analysis layer of ADViSELipidomics — PCA (unsupervised) and PLS-DA (supervised) dimension reduction plus clustering algorithms — to a preprocessed lipid abundance matrix, and generates interactive, customizable plots on lipids and samples (density, spider, scatter, heatmaps).

## When to use

Use when a preprocessed lipidomics data matrix (from LipidSearch/LIQUID outputs or a Metabolomics Workbench experiment) is loaded in the ADViSELipidomics Shiny app and you need unsupervised/supervised dimension reduction, sample clustering, or interactive exploratory visualizations.

## When NOT to use

- The goal is differential abundance testing, internal-standard-based normalization to absolute concentrations, or batch effect correction — these are separate ADViSELipidomics capabilities, not the exploratory dimension-reduction stage
- The input is raw LipidSearch/LIQUID identification output that has not yet been preprocessed into a lipid-by-sample abundance matrix

## Inputs

- Preprocessed lipid abundance matrix (e.g. SummarizedExperiment object) produced by the ADViSELipidomics preprocessing stage
- Sample group labels used as the PLS-DA response
- LipidSearch or LIQUID output files (e.g. Case Study 1 archive), or a Metabolomics Workbench lipidomic experiment downloaded via do_query()

## Outputs

- pca_scores.csv — sample scores per principal component
- plsda_scores.csv — sample scores per latent component
- cluster_assignments.csv — sample-to-cluster mapping
- Interactive, customizable exploratory plots on lipids and samples (density, spider, scatter plots and heatmaps)

## How to apply

Load the Case Study 1 lipidomics data (or a Metabolomics Workbench experiment fetched via the metabolomicsWorkbenchR do_query() function) and launch the app with library("ADViSELipidomics"); run_ADViSELipidomics(). Apply Principal Component Analysis (unsupervised) to the preprocessed lipid abundance matrix and extract sample coordinates along the leading principal components. Fit a PLS-DA model (supervised) on the same matrix using the sample group labels as the response and extract the latent-component score vectors. Cluster the samples on the preprocessed matrix and record the cluster assignment for each sample. Export the results as structured tables — pca_scores.csv (sample scores per component), plsda_scores.csv (sample scores per latent component), and cluster_assignments.csv (sample-to-cluster mapping) — and cross-check them against the interactive plots (density, spider, scatter, heatmaps) shown in the interface, which are all highly customizable.

## Related tools

- **ADViSELipidomics** (R/Shiny application whose exploratory-analysis layer performs PCA, PLS-DA and clustering on the preprocessed data and renders the interactive plots) — https://github.com/ShinyFabio/ADViSELipidomics
- **Shiny** (Framework underlying the ADViSELipidomics app that provides the interactive, customizable plot outputs)

## Examples

```
library("ADViSELipidomics"); run_ADViSELipidomics()
```

## Evaluation signals

- Each output table (pca_scores.csv, plsda_scores.csv, cluster_assignments.csv) exists and is a valid CSV
- Each table contains a score/assignment row for every sample in the preprocessed matrix
- Exported score and cluster values reproduce the exploratory plots displayed by the ADViSELipidomics interface
- PLS-DA latent-component scores are derived using the sample group labels as the supervised response, unlike the unsupervised PCA scores

## Limitations

- The article reports no numeric threshold for validating the exploratory-analysis stage; correctness rests on expert review of plot reproduction
- The interface rendering depends on screen size and resolution; the README advises adjusting zoom (Ctrl/Cmd +/−) and notes the Docker version was tested on Chrome with Firefox potentially problematic
- The article's discussion leaves the batch-correction method name unidentified, which limits traceability of the preprocessing upstream of this stage

## Evidence

- [intro] extracts information by parsing lipid species (using LIPID MAPS classification) and, together with information available on the samples, allows performing several exploratory and statistical analyses: "extracts information by parsing lipid species (using LIPID MAPS classification) and, together with information available on the samples,
allows performing several exploratory and statistical analyses"
- [methods] library("ADViSELipidomics")
run_ADViSELipidomics(): "library("ADViSELipidomics")
run_ADViSELipidomics()"
- [readme] If you want to test the software with a LipidSearch output, here we provide the files used in the Case Study 1: "If you want to test the software with a LipidSearch output, here we
provide the files used in the Case Study 1"
- [readme] ADViSELipidomics can download in real-time suitable selected lipidomic experiments from the online repository using the do_query() function from the {metabolomicsWorkbenchR} package: "ADViSELipidomics can download in real-time suitable selected
lipidomic experiments from the online repository using the `do_query()`
function from the `{metabolomicsWorkbenchR}` package."
