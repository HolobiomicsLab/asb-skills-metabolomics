---
name: ma-volcano-plot-visualization
description: Use when a lipidomics differential analysis (LipidSearch/LIQUID outputs
  or Metabolomics Workbench data) has already been fitted under a defined experimental
  design and comparison, and differentially abundant lipids must be depicted with
  MA and Volcano plots.
when_to_use_negative:
- The experimental design, batch-effect handling, and comparison have not yet been
  defined in the statistical-analysis tab — the plots depend on the fitted differential
  model.
- Input is raw vendor output that has not been parsed with the LIPID MAPS classification
  and normalized (no internal-standard-based concentrations), so no differential model
  exists to plot.
- The goal is exploratory sample-level visualization (e.g. QC or clustering) rather
  than contrast-level differential abundance results.
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_0153
- http://edamontology.org/topic_3315
tools:
- name: ADViSELipidomics
  role: Shiny app that fits the differential abundance model and generates the MA
    plot and Volcano plot from its results, then exports them with the differential
    abundance table
  repo: https://github.com/ShinyFabio/ADViSELipidomics
- name: Shiny
  role: Framework hosting the statistical-analysis tab in which the plots are produced
    and exported
- name: LipidSearch
  role: Source of the lipid abundance matrix input used to fit the model underlying
    the plots (Case Study 1)
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btac706
    title: 'ADViSELipidomics: a workflow for analyzing lipidomics data'
  source_content_hash: sha256:ab23f63fff0270adc00c75424eeb3f7a79c9bc377c2950e3461efeb5534defcf
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ShinyFabio/ADViSELipidomics
  tools:
  - ADViSELipidomics
  - LipidSearch
  - Shiny
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_0153
  - http://edamontology.org/topic_3315
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# ma-volcano-plot-visualization

## Summary

Visualize differential lipid abundance results from ADViSELipidomics as MA plots and Volcano plots, generated from the fitted differential abundance model over the normalized concentration matrix after design and batch-effect handling are set.

## When to use

Use when a lipidomics differential analysis (LipidSearch/LIQUID outputs or Metabolomics Workbench data) has already been fitted under a defined experimental design and comparison, and differentially abundant lipids must be depicted with MA and Volcano plots.

## When NOT to use

- The experimental design, batch-effect handling, and comparison have not yet been defined in the statistical-analysis tab — the plots depend on the fitted differential model.
- Input is raw vendor output that has not been parsed with the LIPID MAPS classification and normalized (no internal-standard-based concentrations), so no differential model exists to plot.
- The goal is exploratory sample-level visualization (e.g. QC or clustering) rather than contrast-level differential abundance results.

## Inputs

- Fitted differential abundance model results (statistics per lipid for the design contrasts)
- Normalized concentration matrix from LipidSearch or LIQUID output (or Metabolomics Workbench data), optionally batch-corrected
- Target-file sample metadata defining the simple or complex experimental design and chosen comparison

## Outputs

- MA plot image file
- Volcano plot image file
- Differential abundance table exported to the output directory

## How to apply

In the ADViSELipidomics Shiny application, first complete the upstream steps: load the lipid abundance matrix with the target-file sample metadata, parse lipid species using the LIPID MAPS classification, define the experimental design in the statistical-analysis tab (simple two-group or complex design with covariates), and optionally apply batch effect correction to the normalized concentration matrix. Then fit the differential abundance model for the design contrasts and generate the MA plot and Volcano plot directly from the model results, so fold-change and significance dimensions are consistent with the fitted contrasts. Optionally rank the lipid list for lipid-set enrichment analysis alongside the plots. Finally export the differential abundance table together with the MA-plot and Volcano-plot image files to the output directory.

## Related tools

- **ADViSELipidomics** (Shiny app that fits the differential abundance model and generates the MA plot and Volcano plot from its results, then exports them with the differential abundance table) — https://github.com/ShinyFabio/ADViSELipidomics
- **Shiny** (Framework hosting the statistical-analysis tab in which the plots are produced and exported)
- **LipidSearch** (Source of the lipid abundance matrix input used to fit the model underlying the plots (Case Study 1))

## Examples

```
library("ADViSELipidomics"); run_ADViSELipidomics()
```

## Evaluation signals

- MA plot and Volcano plot are both generated from the same fitted differential abundance model results, so significance calls match the exported differential abundance table.
- The plotted contrast corresponds to the comparison defined from the target-file metadata under the chosen simple or complex design (including covariates).
- If batch effect correction was applied, the plots are based on the corrected normalized concentration matrix, not the raw one.
- MA-plot and Volcano-plot image files are exported to the output directory alongside the differential abundance table.

## Limitations

- The workflow is described for Case Study 1 (LipidSearch output); results presentation specifics for other sources (LIQUID, Metabolomics Workbench) may vary.
- Batch correction method is applied within ADViSELipidomics but the article discussion notes the method name is not identified in the text ('Batch correction was carried out using ...'), limiting reproducibility of the upstream statistics feeding the plots.
- Dockerized access via Chrome is recommended; the README warns 'Firefox can have some problems', which may affect interactive plot inspection.

## Evidence

- [full_text] MA/Volcano plots derived from fitted model: [evidence span withheld — anchor status: not_found]
- [abstract] Differential analysis in simple and complex designs: "it identifies differentially abundant lipids in simple and complex experimental designs, dealing with batch effect correction."
- [supplementary] Export of plots and tables (source card workflow): [evidence span withheld — anchor status: not_found]
- [readme] Normalization and design definition upstream of plotting: "In the presence of internal lipid standards in the experiment, ADViSELipidomics can normalize the data matrix, providing absolute values of concentration per lipid and sample."
- [intro] Statistical analysis capability in the Shiny app: "extracts information by parsing lipid species (using LIPID MAPS
classification) and, together with information available on the samples,
allows performing several exploratory and statistical analyses"
