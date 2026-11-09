---
name: shiny-app-exploratory-data-processing
description: Use when a lipidomics SummarizedExperiment (from LipidSearch, LIQUID,
  or Metabolomics Workbench inputs) has been preprocessed and you need dimension-reduction
  scores (PCA, PLS-DA), sample cluster assignments, or interactive exploratory plots
  of lipids and samples.
when_to_use_negative:
- The data have not yet been preprocessed by ADViSELipidomics (lipid species parsed
  via LIPID MAPS classification, normalized against internal lipid standards, batch-corrected)
  — run the preprocessing stage first
- The goal is differential-abundance testing in simple or complex experimental designs,
  which is the statistical-analysis stage rather than exploratory analysis
- Input is not a LipidSearch/LIQUID output or a Metabolomics Workbench lipidomic experiment
  supported by the app
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: ADViSELipidomics
  role: 'Shiny application providing the exploratory-analysis layer: PCA (unsupervised)
    and PLS-DA (supervised) dimension reduction, clustering algorithms, and interactive
    customizable plots on lipids and samples'
  repo: https://github.com/ShinyFabio/ADViSELipidomics
- name: Shiny
  role: Web framework in which the exploratory-analysis interface and interactive
    plots are implemented
- name: metabolomicsWorkbenchR
  role: R package whose do_query() function downloads suitable Metabolomics Workbench
    lipidomic experiments in real time as exploratory-analysis input
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btac706
    title: 'ADViSELipidomics: a workflow for analyzing lipidomics data'
  source_content_hash: sha256:91dfd9bdefa6ed3ef756fae559c3c7d9ec28d02b6e76570ad1155052f14b6a3d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ShinyFabio/ADViSELipidomics
  tools:
  - ADViSELipidomics
  - R
  - Shiny
  - metabolomicsWorkbenchR
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8666666666666667
n_claims: 15
---

# shiny-app-exploratory-data-processing

## Summary

Reconstructs the exploratory-analysis layer of the ADViSELipidomics Shiny app, which applies unsupervised PCA, supervised PLS-DA, and several clustering algorithms to the preprocessed lipid abundance matrix, alongside interactive density, spider, scatter and heatmap visualizations of lipids and samples.

## When to use

Use when a lipidomics SummarizedExperiment (from LipidSearch, LIQUID, or Metabolomics Workbench inputs) has been preprocessed and you need dimension-reduction scores (PCA, PLS-DA), sample cluster assignments, or interactive exploratory plots of lipids and samples.

## When NOT to use

- The data have not yet been preprocessed by ADViSELipidomics (lipid species parsed via LIPID MAPS classification, normalized against internal lipid standards, batch-corrected) — run the preprocessing stage first
- The goal is differential-abundance testing in simple or complex experimental designs, which is the statistical-analysis stage rather than exploratory analysis
- Input is not a LipidSearch/LIQUID output or a Metabolomics Workbench lipidomic experiment supported by the app

## Inputs

- Preprocessed lipid abundance matrix stored in a SummarizedExperiment object (from LipidSearch or LIQUID outputs, or Metabolomics Workbench experiments fetched via do_query() from {metabolomicsWorkbenchR})
- Sample group labels for the supervised PLS-DA response

## Outputs

- pca_scores.csv — sample scores per principal component
- plsda_scores.csv — sample scores per latent component
- cluster_assignments.csv — sample-to-cluster mapping
- Interactive exploratory plots (density, spider, scatter, heatmap) of lipids and samples

## How to apply

Load Case Study 1 lipidomics data and launch the app with library("ADViSELipidomics"); run_ADViSELipidomics(), or run the Docker image shinyfabio/adviselipidomics:latest and open http://localhost:80/. Apply Principal Component Analysis (unsupervised dimension reduction) to the preprocessed lipid abundance matrix and extract sample coordinates along the leading principal components; fit a PLS-DA model (supervised dimension reduction) using the sample group labels as the response and extract latent-component score vectors; then cluster the samples on the same matrix and record each sample's cluster assignment. Export the plot data as structured tables: pca_scores.csv (sample scores per component), plsda_scores.csv (sample scores per latent component), and cluster_assignments.csv (sample-to-cluster mapping). The app also generates interactive plots on lipids and samples — density, spider, scatter plots and heatmaps — that are highly customizable, so exported values should reproduce what the interface displays.

## Related tools

- **ADViSELipidomics** (Shiny application providing the exploratory-analysis layer: PCA (unsupervised) and PLS-DA (supervised) dimension reduction, clustering algorithms, and interactive customizable plots on lipids and samples) — https://github.com/ShinyFabio/ADViSELipidomics
- **Shiny** (Web framework in which the exploratory-analysis interface and interactive plots are implemented)
- **metabolomicsWorkbenchR** (R package whose do_query() function downloads suitable Metabolomics Workbench lipidomic experiments in real time as exploratory-analysis input)

## Examples

```
library("ADViSELipidomics"); run_ADViSELipidomics()
```

## Evaluation signals

- Each exported table (pca_scores.csv, plsda_scores.csv, cluster_assignments.csv) exists and is a valid CSV
- Every table contains a score or assignment row for every sample in the preprocessed matrix (row count matches sample count)
- Exported PCA and PLS-DA score values reproduce the exploratory plots displayed by the ADViSELipidomics interface (expert visual review, since the article reports no numeric threshold for this stage)
- Cluster_assignments.csv yields a one-to-one sample-to-cluster mapping with no missing samples
- PLS-DA latent components are fitted against the sample group labels, so score vectors separate by group as shown in the app's interactive plots

## Limitations

- The article reports no numeric threshold for validating the exploratory stage, so correctness is judged by expert review against the app's displayed plots
- The Shiny interface's appearance depends on screen size and monitor resolution, and may differ from how it was designed; users are advised to adjust zoom with Ctrl/Command and +/-
- Dockerized version was tested on Chrome; Firefox can have some problems
- Power to detect non-linear drift is low in replicate QC samples because the variance is very low, which limits exploratory QC-plateau interpretation

## Evidence

- [intro] Exploratory analysis methods and interactive plots: "extracts information by parsing lipid species (using LIPID MAPS classification) and, together with information available on the samples, allows performing several exploratory and statistical analyses"
- [readme] Shiny app scope statement: "ADViSELipidomics is a novel Shiny app for the preprocessing, analysis,
and visualization of lipidomics data"
- [readme] Launch command: "Once the installation is completed, run:

``` r
library("ADViSELipidomics")
run_ADViSELipidomics()"
- [methods] Docker access route: "docker run -p 80:80 shinyfabio/adviselipidomics:latest"
