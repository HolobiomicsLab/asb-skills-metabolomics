---
name: differential-lipid-abundance-testing
description: Use when a normalized lipid abundance matrix (from LipidSearch/LIQUID
  output or Metabolomics Workbench import) is available and the goal is to find differentially
  abundant lipids between groups defined in a target-file metadata design, optionally
  with covariates or batch correction.
when_to_use_negative:
- Input is still raw LipidSearch/LIQUID output or an unnormalized intensity matrix
  — parse lipid species (LIPID MAPS classification) and normalize with internal lipid
  standards first.
- No experimental design or group metadata has been defined in the target file — the
  design must be set before fitting the differential abundance model.
- The goal is exploratory visualization or lipid-species parsing rather than statistical
  testing between conditions.
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3315
- http://edamontology.org/topic_0154
tools:
- name: ADViSELipidomics
  role: Shiny app providing the statistical-analysis tab for design definition, batch
    effect correction, differential abundance model fitting, MA/volcano plot generation,
    and lipid-set enrichment
  repo: https://github.com/ShinyFabio/ADViSELipidomics
- name: Shiny
  role: Framework in which the ADViSELipidomics differential-analysis interface is
    implemented
  repo: https://github.com/ShinyFabio/ADViSELipidomics
- name: LipidSearch
  role: Source of the lipid identification and quantification output files whose abundance
    matrix feeds the differential analysis (Case Study 1)
provenance:
  source_task_ids:
  - task_004
  - task_006
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btac706
    title: 'ADViSELipidomics: a workflow for analyzing lipidomics data'
  source_content_hash: sha256:862308616305357cec723334c0fbda78b855ff24ca45c40291460a738885c98c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ShinyFabio/ADViSELipidomics
  tools:
  - ADViSELipidomics
  - LipidSearch
  - Shiny
  - metabolomicsWorkbenchR
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_3315
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 14
---

# differential-lipid-abundance-testing

## Summary

Identify differentially abundant lipids between experimental conditions using ADViSELipidomics' statistical-analysis module, which fits models over the normalized concentration matrix under simple or complex designs and visualizes results as MA and volcano plots with lipid-set enrichment.

## When to use

Use when a normalized lipid abundance matrix (from LipidSearch/LIQUID output or Metabolomics Workbench import) is available and the goal is to find differentially abundant lipids between groups defined in a target-file metadata design, optionally with covariates or batch correction.

## When NOT to use

- Input is still raw LipidSearch/LIQUID output or an unnormalized intensity matrix — parse lipid species (LIPID MAPS classification) and normalize with internal lipid standards first.
- No experimental design or group metadata has been defined in the target file — the design must be set before fitting the differential abundance model.
- The goal is exploratory visualization or lipid-species parsing rather than statistical testing between conditions.

## Inputs

- Normalized lipid concentration matrix (lipids x samples) from ADViSELipidomics preprocessing/normalization
- Target-file sample metadata defining groups, covariates, and batch variables
- Experimental design specification (simple two-group or complex design with covariates)
- Optionally, batch-corrected normalized concentration matrix

## Outputs

- Differential abundance table of differentially abundant lipids between design contrasts
- MA plot image file
- Volcano plot image file
- Lipid-set enrichment analysis results table over the ranked lipid list

## How to apply

After preprocessing and (optionally) internal-standard normalization yield a normalized concentration matrix per lipid and sample, define the experimental design in the statistical-analysis tab using the target-file sample metadata, supporting either a simple two-group design (e.g., monocytes vs iDCs, healthy vs diabetic serum) or a complex design with covariates. If the experiment spans batches, apply the built-in batch effect correction to the normalized matrix before fitting. Specify the comparison/contrast of interest, fit the differential abundance model, and inspect results through MA plots and volcano plots generated from the model output. Finally, run the lipid-set enrichment analysis over the ranked lipid list, and export the differential abundance table, the MA-plot and volcano-plot images, and the enrichment results table to the output directory.

## Related tools

- **ADViSELipidomics** (Shiny app providing the statistical-analysis tab for design definition, batch effect correction, differential abundance model fitting, MA/volcano plot generation, and lipid-set enrichment) — https://github.com/ShinyFabio/ADViSELipidomics
- **Shiny** (Framework in which the ADViSELipidomics differential-analysis interface is implemented) — https://github.com/ShinyFabio/ADViSELipidomics
- **LipidSearch** (Source of the lipid identification and quantification output files whose abundance matrix feeds the differential analysis (Case Study 1))

## Examples

```
library("ADViSELipidomics"); run_ADViSELipidomics()
```

## Evaluation signals

- The monocytes-vs-iDCs comparison on Case Study 1 (LipidSearch output, internal-standard normalized) reproduces the Case Study 1 results in the article's Supplementary Material, matching normalized concentration values per lipids and samples.
- The healthy-vs-diabetic serum differential lipid list and statistics on the Metabolomics Workbench import match the Case Study 2 results reported in the paper (qualitative acceptance check; no quantitative threshold is given).
- MA and volcano plots are successfully generated from the fitted model results and exported alongside the differential abundance table.
- The lipid-set enrichment results table is produced over the ranked lipid list and exported to the output directory.
- The design supports both a simple two-group comparison and a complex design with covariates, with batch effect correction applied only when the normalized matrix spans batches.

## Limitations

- Power to detect non-linear drift in QC samples is low: the 2D pooled-distribution MAD pattern is not visible in replicate (n=1 vs n=2) QC samples because variance is very low.
- The specific batch correction method applied is not identified in the paper (the method name is not given).
- Metabolomics Workbench import requires an internet connection and is limited to suitable selected lipidomic experiments via do_query().

## Evidence

- [supplementary] Differential analysis presentation: [evidence span withheld — anchor status: not_found]
- [abstract] Designs and batch correction: "it identifies differentially abundant lipids in simple and complex experimental designs, dealing with batch effect correction."
- [abstract] Normalized matrix prerequisite: "When the experiment includes internal lipid standards, ADViSELipidomics can normalize the data matrix, providing normalized concentration values per lipids and samples."
- [supplementary] Case study 1 validation: [evidence span withheld — anchor status: not_found]
- [supplementary] Case study 2 qualitative check: [evidence span withheld — anchor status: not_found]
- [discussion] Batch correction limitation: "Batch correction was carried out using "
