---
name: experimental-design-specification-from-target-metadata
description: Use when a lipidomics abundance matrix (LipidSearch/LIQUID output or
  Metabolomics Workbench data) is loaded with its target-file sample metadata and
  a differential abundance model must be defined via design matrix and contrasts.
when_to_use_negative:
- The data are raw vendor peak lists not yet processed into a lipid abundance matrix
  — preprocessing and normalization come first in the ADViSELipidomics workflow
- The experiment lacks a target-file or sample metadata, so no groups or covariates
  can be specified for a design
edam_operation: http://edamontology.org/operation_3664
edam_topics:
- http://edamontology.org/topic_3314
- http://edamontology.org/topic_0203
tools:
- name: ADViSELipidomics
  role: Shiny app whose statistical-analysis tab is used to define the experimental
    design (simple or complex) from target-file metadata and fit the differential
    abundance model with batch effect correction
  repo: https://github.com/ShinyFabio/ADViSELipidomics
- name: Shiny
  role: Framework implementing the ADViSELipidomics interface in which the design
    is specified
  repo: https://github.com/ShinyFabio/ADViSELipidomics
- name: LipidSearch
  role: Source of the lipid identification/quantification output files whose sample
    metadata (target-file) defines the design
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btac706
    title: 'ADViSELipidomics: a workflow for analyzing lipidomics data'
  source_content_hash: sha256:770a6fc243fa24d960a529ac252437949ab99487daa41bf4b28fa700b47694fc
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ShinyFabio/ADViSELipidomics
  tools:
  - ADViSELipidomics
  - LipidSearch
  - Shiny
  edam_operation: http://edamontology.org/operation_3664
  edam_topics:
  - http://edamontology.org/topic_3314
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# experimental-design-specification-from-target-metadata

## Summary

Specify the statistical experimental design for differential lipid abundance analysis in ADViSELipidomics by combining LIPID MAPS-parsed lipid species information with sample metadata from the target-file, supporting both simple two-group designs and complex designs with covariates.

## When to use

Use when a lipidomics abundance matrix (LipidSearch/LIQUID output or Metabolomics Workbench data) is loaded with its target-file sample metadata and a differential abundance model must be defined via design matrix and contrasts.

## When NOT to use

- The data are raw vendor peak lists not yet processed into a lipid abundance matrix — preprocessing and normalization come first in the ADViSELipidomics workflow
- The experiment lacks a target-file or sample metadata, so no groups or covariates can be specified for a design

## Inputs

- LipidSearch or LIQUID output files containing the lipid abundance matrix
- Target-file with per-sample metadata (groups, covariates, batch information)
- Lipid species annotations parsed via LIPID MAPS classification
- Optionally, normalized concentration values from internal-standard-based normalization

## Outputs

- Defined experimental design (design matrix and comparison/contrast) in the statistical-analysis tab
- Differentially abundant lipid results between design contrasts
- MA plot and volcano plot from the model results
- Lipid-set enrichment results over the ranked lipid list
- Exportable differential abundance table and plot image files to the output directory

## How to apply

Load the Case Study 1 LipidSearch output files (lipid abundance matrix and target-file sample metadata) into the ADViSELipidomics Shiny application. Parse lipid species using the LIPID MAPS classification so lipid annotations can be combined with the sample information in the target-file. In the statistical-analysis tab, define the experimental design: choose a simple two-group design or a complex design that includes covariates, which determines the design matrix used for model fitting. Optionally enable the built-in batch effect correction on the normalized concentration matrix before fitting. Once design, batch handling, and comparison are set, ADViSELipidomics fits the differential abundance model and surfaces results as MA plots, volcano plots, and lipid-set enrichment over the ranked lipid list.

## Related tools

- **ADViSELipidomics** (Shiny app whose statistical-analysis tab is used to define the experimental design (simple or complex) from target-file metadata and fit the differential abundance model with batch effect correction) — https://github.com/ShinyFabio/ADViSELipidomics
- **Shiny** (Framework implementing the ADViSELipidomics interface in which the design is specified) — https://github.com/ShinyFabio/ADViSELipidomics
- **LipidSearch** (Source of the lipid identification/quantification output files whose sample metadata (target-file) defines the design)

## Examples

```
library("ADViSELipidomics"); run_ADViSELipidomics()
```

## Evaluation signals

- The design supports both a simple two-group layout and a complex design with covariates before model fitting
- Batch effect correction can be toggled on the normalized concentration matrix prior to fitting the model
- After fitting, MA plots and volcano plots are generated from the model results and lipid-set enrichment runs over the ranked lipid list
- The differential abundance table, MA/volcano plot image files, and lipid-set enrichment results table can be exported to the output directory

## Limitations

- ADViSELipidomics copes only with outputs from LipidSearch and LIQUID and with data from the Metabolomics Workbench; other input formats are out of scope
- Normalization to absolute concentrations requires internal lipid standards to be present in the experiment
- Interface layout depends on screen size and resolution; the application may render differently than intended (adjust zoom, and prefer Chrome over Firefox for the Docker version)

## Evidence

- [abstract] Simple and complex design support: "it identifies differentially abundant lipids in simple and complex experimental designs, dealing with batch effect correction."
- [intro] Design definition from parsed lipids and sample info: "extracts information by parsing lipid species (using LIPID MAPS classification) and, together with information available on the samples, allows performing several exploratory and statistical analyses"
- [readme] Normalization precondition for concentration values: "In the presence of internal lipid standards in the experiment, ADViSELipidomics can normalize the data matrix, providing absolute values of concentration per lipid and sample."
- [supplementary] Case Study 1 files used to exercise the workflow: "If you want to test the software with a LipidSearch output, here we provide the files used in the Case Study 1"
