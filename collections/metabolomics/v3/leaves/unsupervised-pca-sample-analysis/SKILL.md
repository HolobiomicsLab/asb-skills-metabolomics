---
name: unsupervised-pca-sample-analysis
description: Use when a preprocessed lipidomics data matrix (from LipidSearch or LIQUID
  output, or Metabolomics Workbench data loaded via do_query()) is available in ADViSELipidomics
  and the goal is an unsupervised, label-free overview of sample structure and dominant
  axes of variation.
when_to_use_negative:
- The analysis goal is supervised discrimination using known sample group labels —
  use the PLS-DA dimension-reduction option instead, which uses group labels as the
  response
- The input is raw LipidSearch or LIQUID output that has not yet gone through ADViSELipidomics'
  preprocessing (lipid-species parsing via LIPID MAPS classification and, where internal
  lipid standards are present, normalization to absolute concentrations)
- The data contain batch effects that need correction — address the differential-abundance/batch-correction
  layer first, since PCA is an exploratory method on the stored preprocessed matrix
tools:
- name: ADViSELipidomics
  role: Shiny application whose exploratory-analysis layer provides the PCA (unsupervised)
    option on the preprocessed lipidomics data and the interactive sample/lipid plots
    used to inspect PC scores
  repo: https://github.com/ShinyFabio/ADViSELipidomics
- name: Shiny
  role: R framework implementing the ADViSELipidomics interface in which the PCA scores
    and interactive plots are generated
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btac706
    title: 'ADViSELipidomics: a workflow for analyzing lipidomics data'
  source_content_hash: sha256:e0a4990173e66e7b71ccc9c7bbfc0a824d84fdaeda5cabf95b536b10739d6f78
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
triage_status: critique
yes_rate: 0.5384615384615384
n_claims: 13
---

# unsupervised-pca-sample-analysis

## Summary

Apply unsupervised Principal Component Analysis to the preprocessed lipid abundance matrix stored in ADViSELipidomics' SummarizedExperiment object, extracting sample coordinates along the leading principal components as an exploratory overview of sample structure.

## When to use

Use when a preprocessed lipidomics data matrix (from LipidSearch or LIQUID output, or Metabolomics Workbench data loaded via do_query()) is available in ADViSELipidomics and the goal is an unsupervised, label-free overview of sample structure and dominant axes of variation.

## When NOT to use

- The analysis goal is supervised discrimination using known sample group labels — use the PLS-DA dimension-reduction option instead, which uses group labels as the response
- The input is raw LipidSearch or LIQUID output that has not yet gone through ADViSELipidomics' preprocessing (lipid-species parsing via LIPID MAPS classification and, where internal lipid standards are present, normalization to absolute concentrations)
- The data contain batch effects that need correction — address the differential-abundance/batch-correction layer first, since PCA is an exploratory method on the stored preprocessed matrix

## Inputs

- Preprocessed lipid abundance matrix stored as a SummarizedExperiment object in ADViSELipidomics (derived from LipidSearch or LIQUID output, or Metabolomics Workbench data fetched via metabolomicsWorkbenchR::do_query())
- Sample metadata available on the samples (used for plotting/coloring; not required as a response by unsupervised PCA)

## Outputs

- Sample score vectors along the leading principal components
- Structured CSV table pca_scores.csv with one score row per sample per component
- Interactive PCA plot of samples in the Shiny interface

## How to apply

Launch the app with library("ADViSELipidomics"); run_ADViSELipidomics() (or the Docker image on localhost:80) and load the preprocessed data from the Case Study 1 files or a Metabolomics Workbench experiment. In the exploratory-analysis layer, select Principal Component Analysis, which is the unsupervised dimension-reduction method offered alongside the supervised PLS-DA; PCA requires no group labels as response. Extract the sample score vectors along the leading principal components from the stored SummarizedExperiment object and export them as a structured table (pca_scores.csv, one score row per sample per component). Cross-check the exported scores against the app's interactive plots on lipids and samples (density, spider, scatter plots and heatmaps), which are all interactive and highly customizable; the article reports no numeric threshold for this exploratory stage, so validation is by expert review of plot agreement rather than a cutoff.

## Related tools

- **ADViSELipidomics** (Shiny application whose exploratory-analysis layer provides the PCA (unsupervised) option on the preprocessed lipidomics data and the interactive sample/lipid plots used to inspect PC scores) — https://github.com/ShinyFabio/ADViSELipidomics
- **Shiny** (R framework implementing the ADViSELipidomics interface in which the PCA scores and interactive plots are generated)

## Examples

```
library("ADViSELipidomics"); run_ADViSELipidomics()
```

## Evaluation signals

- pca_scores.csv exists and is a valid CSV with a score row for every sample present in the preprocessed lipid abundance matrix
- Exported PC score values reproduce the interactive PCA plot shown by the ADViSELipidomics interface (expert visual review, as the article reports no numeric threshold for this stage)
- The analysis used no group labels — confirm it is the unsupervised PCA option, not PLS-DA, which would fit latent components against sample group labels as the response
- Score vectors correspond to the leading principal components of the preprocessed matrix rather than raw LipidSearch/LIQUID intensities

## Limitations

- The article reports no numeric validation threshold for the exploratory-analysis stage; correctness is judged by expert review that exported values reproduce the plots shown in the interface
- The ADViSELipidomics interface appearance depends on monitor size and resolution; zoom may need adjustment (Ctrl +/- on Windows/Linux, Command +/- on Mac)
- The Docker version is tested with Chrome; Firefox can have some problems rendering the Shiny interface

## Evidence

- [intro] intro quote: "extracts information by parsing lipid species (using LIPID MAPS classification) and, together with information available on the samples"
- [methods] methods launch quote: "library("ADViSELipidomics")
run_ADViSELipidomics()"
- [readme] readme overview quote: "ADViSELipidomics is a novel Shiny app for the preprocessing, analysis, and visualization of lipidomics data"
