---
name: plsda-supervised-group-discrimination
description: Use when a preprocessed lipidomics data matrix (e.g. from LipidSearch,
  LIQUID, or Metabolomics Workbench input) has sample group labels and the goal is
  supervised group discrimination via latent-component scores, as in ADViSELipidomics'
  exploratory-analysis stage.
when_to_use_negative:
- When the experiment has no sample group labels — PCA (the unsupervised dimension-reduction
  option) is the appropriate choice instead of supervised PLS-DA
- When the input is raw LipidSearch/LIQUID output that has not yet passed the app's
  preprocessing/normalization step (internal-standard normalization to absolute concentrations)
- When the aim is differential-abundance testing with batch effect correction rather
  than exploratory group discrimination — use the app's statistical-analysis layer
  instead
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: ADViSELipidomics
  role: R/Shiny package whose exploratory-analysis layer performs PLS-DA (supervised)
    on the preprocessed SummarizedExperiment lipidomics data and renders the interactive
    score plots
  repo: https://github.com/ShinyFabio/ADViSELipidomics
- name: Shiny
  role: Web framework hosting the ADViSELipidomics app through which the interactive,
    customizable PLS-DA outputs are generated
  repo: https://github.com/ShinyFabio/ADViSELipidomics
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btac706
    title: 'ADViSELipidomics: a workflow for analyzing lipidomics data'
  source_content_hash: sha256:4301fd7ed6c9d0b336fb45bf28cf5bb4e70bef70ef66c7e6b0cbb08a498746af
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ShinyFabio/ADViSELipidomics
  tools:
  - ADViSELipidomics
  - R
  - Shiny
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# plsda-supervised-group-discrimination

## Summary

Supervised dimension-reduction analysis within the ADViSELipidomics Shiny workflow: a PLS-DA model is fitted on the preprocessed lipid abundance matrix using sample group labels as the response, alongside PCA and clustering, to discriminate and visualize group separation of lipidomics samples.

## When to use

Use when a preprocessed lipidomics data matrix (e.g. from LipidSearch, LIQUID, or Metabolomics Workbench input) has sample group labels and the goal is supervised group discrimination via latent-component scores, as in ADViSELipidomics' exploratory-analysis stage.

## When NOT to use

- When the experiment has no sample group labels — PCA (the unsupervised dimension-reduction option) is the appropriate choice instead of supervised PLS-DA
- When the input is raw LipidSearch/LIQUID output that has not yet passed the app's preprocessing/normalization step (internal-standard normalization to absolute concentrations)
- When the aim is differential-abundance testing with batch effect correction rather than exploratory group discrimination — use the app's statistical-analysis layer instead

## Inputs

- Preprocessed lipid abundance matrix stored as a SummarizedExperiment object (from LipidSearch or LIQUID output files, or a Metabolomics Workbench experiment retrieved via metabolomicsWorkbenchR::do_query())
- Sample metadata with group labels used as the PLS-DA response

## Outputs

- Sample score vectors per PLS-DA latent component (exportable as plsda_scores.csv)
- Interactive PLS-DA score plot within the ADViSELipidomics Shiny interface, customizable alongside density, spider, scatter and heatmap views

## How to apply

Load the Case Study lipidomics data into R, launch the app with library("ADViSELipidomics"); run_ADViSELipidomics(), and use the exploratory-analysis layer that operates on the stored SummarizedExperiment object. Fit the PLS-DA model (the supervised dimension-reduction option) on the preprocessed lipid abundance matrix using the sample group labels as the response, and extract the sample score vectors along the latent components. ADViSELipidomics also offers unsupervised PCA and several clustering algorithms on the same matrix, so PLS-DA should be selected specifically when class labels are available and discrimination is the goal. All exploratory outputs are rendered as interactive, highly customizable plots on lipids and samples (density, spider, scatter plots and heatmaps); the PLS-DA scores can additionally be exported as a structured CSV table (plsda_scores.csv) with one score row per sample, alongside pca_scores.csv and cluster_assignments.csv. Because the article reports no numeric performance threshold for this stage, validation of the PLS-DA fit is by expert review against the plots shown in the interface.

## Related tools

- **ADViSELipidomics** (R/Shiny package whose exploratory-analysis layer performs PLS-DA (supervised) on the preprocessed SummarizedExperiment lipidomics data and renders the interactive score plots) — https://github.com/ShinyFabio/ADViSELipidomics
- **Shiny** (Web framework hosting the ADViSELipidomics app through which the interactive, customizable PLS-DA outputs are generated) — https://github.com/ShinyFabio/ADViSELipidomics

## Examples

```
library("ADViSELipidomics"); run_ADViSELipidomics()
```

## Evaluation signals

- plsda_scores.csv exists as a valid CSV and contains a score row for every sample present in the preprocessed lipid abundance matrix
- The exported latent-component scores reproduce the PLS-DA plot shown in the ADViSELipidomics interface (expert review, since the article reports no numeric threshold for this stage)
- The same preprocessed matrix yields consistent sample sets across the companion pca_scores.csv and cluster_assignments.csv tables
- Group separation in the score plot aligns with the sample labels supplied from the samples information (supervised model, not unsupervised PCA structure)

## Limitations

- The article reports no numeric performance threshold for the exploratory stage; correctness of the PLS-DA is assessed by expert review against interface plots rather than a quantitative criterion
- The interface layout can differ from how it was designed depending on monitor resolution/zoom, which may affect interactive plot inspection
- PLS-DA applies to the stored preprocessed data; it cannot be run on raw LipidSearch/LIQUID outputs before the app's preprocessing and, where applicable, internal-standard normalization

## Evidence

- [abstract] Shiny app scope (abstract): "ADViSELipidomics is a novel Shiny app for preprocessing, analyzing and visualizing lipidomics data."
- [intro] Exploratory and statistical analyses on samples (intro): "extracts information by parsing lipid species (using LIPID MAPS classification) and, together with information available on the samples, allows performing several exploratory and statistical analyses"
- [readme] LipidSearch/LIQUID and Metabolomics Workbench inputs (readme): "It copes with the outputs from LipidSearch and LIQUID for lipid identification and quantification, and with data available from the Metabolomics Workbench."
- [methods] Launch invocation (methods): "library("ADViSELipidomics")
run_ADViSELipidomics()"
