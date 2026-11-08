---
name: sample-clustering-analysis
description: Use when a SummarizedExperiment of preprocessed lipid abundances (from
  LipidSearch, LIQUID, or Metabolomics Workbench input) needs unsupervised sample
  grouping as part of an exploratory lipidomics analysis, e.
when_to_use_negative:
- The data are raw vendor/LipidSearch output that has not passed ADViSELipidomics
  preprocessing (lipid-species parsing with LIPID MAPS classification and, where internal
  standards are present, normalization to absolute concentrations) — cluster first
  on the preprocessed matrix.
- The goal is differential-abundance testing or batch effect correction rather than
  exploratory sample grouping — those are separate stages of the ADViSELipidomics
  workflow.
- A supervised classification model is the actual target — use PLS-DA with sample
  group labels instead of unsupervised clustering.
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3314
- http://edamontology.org/topic_0091
tools:
- name: ADViSELipidomics
  role: Shiny app whose exploratory-analysis layer performs PCA, PLS-DA and clustering
    on the preprocessed SummarizedExperiment lipidomics data and renders the interactive
    plots
  repo: https://github.com/ShinyFabio/ADViSELipidomics
- name: Shiny
  role: Framework in which ADViSELipidomics' interactive, customizable clustering
    and visualization plots are implemented
  repo: https://github.com/ShinyFabio/ADViSELipidomics
- name: metabolomicsWorkbenchR
  role: Provides do_query() to download suitable lipidomic experiments from Metabolomics
    Workbench as clustering input data
- name: R
  role: Language in which ADViSELipidomics is implemented and installed (R > 4.0)
    via devtools::install_github
  repo: https://github.com/ShinyFabio/ADViSELipidomics
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btac706
    title: 'ADViSELipidomics: a workflow for analyzing lipidomics data'
  source_content_hash: sha256:b5413420480edfda311b053ed67f2999a478001df598834a14076dc9642ebc10
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ShinyFabio/ADViSELipidomics
  tools:
  - ADViSELipidomics
  - R
  - Shiny
  - metabolomicsWorkbenchR
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3314
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# sample-clustering-analysis

## Summary

Applies the clustering stage of ADViSELipidomics' exploratory-analysis layer to group samples of a preprocessed lipidomics matrix (LipidSearch/LIQUID outputs or Metabolomics Workbench data parsed via LIPID MAPS classification), alongside PCA and PLS-DA, and exports per-sample cluster assignments for downstream visualization.

## When to use

Use when a SummarizedExperiment of preprocessed lipid abundances (from LipidSearch, LIQUID, or Metabolomics Workbench input) needs unsupervised sample grouping as part of an exploratory lipidomics analysis, e.g. to check whether samples separate by experimental group before differential-abundance testing.

## When NOT to use

- The data are raw vendor/LipidSearch output that has not passed ADViSELipidomics preprocessing (lipid-species parsing with LIPID MAPS classification and, where internal standards are present, normalization to absolute concentrations) — cluster first on the preprocessed matrix.
- The goal is differential-abundance testing or batch effect correction rather than exploratory sample grouping — those are separate stages of the ADViSELipidomics workflow.
- A supervised classification model is the actual target — use PLS-DA with sample group labels instead of unsupervised clustering.

## Inputs

- Preprocessed lipid abundance matrix stored in a SummarizedExperiment object (from LipidSearch or LIQUID output files, or a Metabolomics Workbench experiment fetched via do_query() from metabolomicsWorkbenchR)
- Sample metadata / group labels accompanying the lipidomics experiment

## Outputs

- cluster_assignments.csv — sample-to-cluster mapping with one assignment row per sample
- pca_scores.csv — sample scores per principal component (companion PCA output)
- plsda_scores.csv — sample scores per latent component (companion PLS-DA output)
- Interactive cluster/heatmap plots rendered in the Shiny interface

## How to apply

Load the Case Study or user data into ADViSELipidomics and launch it with library("ADViSELipidomics"); run_ADViSELipidomics() (or the Docker image shinyfabio/adviselipidomics:latest on localhost:80). In the exploratory-analysis layer, run clustering on the stored, preprocessed lipid abundance matrix together with the two dimension-reduction options offered by the app — PCA (unsupervised) and PLS-DA (supervised, using sample group labels as the response). Record the cluster assignment for every sample in the matrix and export it as a structured table (cluster_assignments.csv: sample-to-cluster mapping), alongside pca_scores.csv and plsda_scores.csv when those analyses are also run. Validate that every sample in the preprocessed matrix has a cluster assignment and that the exported values reproduce the interactive plots (density, spider, scatter, heatmaps) shown by the ADViSELipidomics interface; the article reports no numeric threshold for this stage, so expert visual review is the criterion.

## Related tools

- **ADViSELipidomics** (Shiny app whose exploratory-analysis layer performs PCA, PLS-DA and clustering on the preprocessed SummarizedExperiment lipidomics data and renders the interactive plots) — https://github.com/ShinyFabio/ADViSELipidomics
- **Shiny** (Framework in which ADViSELipidomics' interactive, customizable clustering and visualization plots are implemented) — https://github.com/ShinyFabio/ADViSELipidomics
- **metabolomicsWorkbenchR** (Provides do_query() to download suitable lipidomic experiments from Metabolomics Workbench as clustering input data)
- **R** (Language in which ADViSELipidomics is implemented and installed (R > 4.0) via devtools::install_github) — https://github.com/ShinyFabio/ADViSELipidomics

## Examples

```
library("ADViSELipidomics"); run_ADViSELipidomics()
```

## Evaluation signals

- cluster_assignments.csv exists and is a valid CSV with exactly one assignment row per sample in the preprocessed SummarizedExperiment matrix — no missing samples.
- pca_scores.csv and plsda_scores.csv (when generated) likewise contain a score row for every sample and the expected number of components/latent variables as columns.
- Exported score and assignment values reproduce the exploratory plots shown by the ADViSELipidomics Shiny interface when re-plotted.
- Cluster structure is consistent with the interactive heatmaps and scatter plots on lipids and samples in the app (expert visual review; the article reports no numeric threshold for this stage).

## Limitations

- The article reports no numeric validation threshold for the exploratory stage; correctness is judged by expert review against the interactive plots.
- The app's interface layout depends on monitor resolution/zoom, which can complicate visual inspection (Ctrl/Cmd +/- zoom advice in the README).
- Clustering operates on data already shaped by upstream choices (LipidSearch/LIQUID quantification, LIPID MAPS parsing, and internal-standard normalization when standards are present), so results inherit any upstream artifacts.

## Evidence

- [supplementary] The exploratory-analysis stage offers two dimension-reduction methods on the stored data — Principal Component Analysis (unsupervised) and Partial Least Square–Discriminant Analysis (supervised) — plus several clustering algorithms: [evidence span withheld — anchor status: not_found]
- [methods] library and run invocation: "library("ADViSELipidomics")
run_ADViSELipidomics()"
- [intro] exploratory analyses statement: "extracts information by parsing lipid species (using LIPID MAPS
classification) and, together with information available on the samples,
allows performing several exploratory and statistical analyses"
- [readme] readme app description: "ADViSELipidomics is a novel Shiny app for the preprocessing, analysis,
and visualization of lipidomics data."
- [supplementary] Metabolomics Workbench input path: "ADViSELipidomics can download in real-time suitable selected
lipidomic experiments from the online repository using the `do_query()`
function from the `{metabolomicsWorkbenchR}` package"
