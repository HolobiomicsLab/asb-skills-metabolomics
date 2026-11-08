---
name: lipidomics-batch-effect-correction
description: Use when a lipidomics abundance matrix (LipidSearch/LIQUID output or
  Metabolomics Workbench data) spans multiple acquisition batches and the user wants
  corrected values feeding the differential lipid abundance analysis (MA/volcano plots).
when_to_use_negative:
- The experiment contains no batch structure (single acquisition batch), since the
  correction is an optional step applied only when batch effects are present in the
  design metadata.
- Internal lipid standards are absent, so the concentration matrix cannot be normalized
  before the correction and downstream absolute-concentration outputs would be unavailable.
- The input is raw vendor instrument output rather than a processed lipid abundance
  matrix from LipidSearch, LIQUID, or Metabolomics Workbench, which ADViSELipidomics
  does not handle.
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: ADViSELipidomics
  role: Shiny app that provides the built-in batch effect correction applied to the
    normalized concentration matrix within the statistical-analysis tab
  repo: https://github.com/ShinyFabio/ADViSELipidomics
- name: Shiny
  role: Framework implementing the ADViSELipidomics interface in which the batch correction
    option is exposed
  repo: https://github.com/ShinyFabio/ADViSELipidomics
- name: LipidSearch
  role: Source of the lipid identification/quantification output files whose abundance
    matrix undergoes batch correction in Case Study 1
- name: LIQUID
  role: Alternative source of lipid identification/quantification outputs accepted
    by the app for batch-corrected analysis
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btac706
    title: 'ADViSELipidomics: a workflow for analyzing lipidomics data'
  source_content_hash: sha256:1f7e95b0835e76e44c82e2623e90e2e7378e2fd1673ae40facc68667a3b43408
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ShinyFabio/ADViSELipidomics
  tools:
  - ADViSELipidomics
  - LIQUID
  - LipidSearch
  - Shiny
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# lipidomics-batch-effect-correction

## Summary

Optional built-in step in the ADViSELipidomics Shiny app that corrects batch effects in the normalized lipid concentration matrix (from LipidSearch/LIQUID outputs or Metabolomics Workbench experiments) before differential abundance testing, improving comparability of lipids across acquisition batches.

## When to use

Use when a lipidomics abundance matrix (LipidSearch/LIQUID output or Metabolomics Workbench data) spans multiple acquisition batches and the user wants corrected values feeding the differential lipid abundance analysis (MA/volcano plots).

## When NOT to use

- The experiment contains no batch structure (single acquisition batch), since the correction is an optional step applied only when batch effects are present in the design metadata.
- Internal lipid standards are absent, so the concentration matrix cannot be normalized before the correction and downstream absolute-concentration outputs would be unavailable.
- The input is raw vendor instrument output rather than a processed lipid abundance matrix from LipidSearch, LIQUID, or Metabolomics Workbench, which ADViSELipidomics does not handle.

## Inputs

- Lipid abundance matrix from LipidSearch output (Case Study 1 format)
- Lipid abundance matrix from LIQUID output
- Lipidomic experiment downloaded from Metabolomics Workbench via metabolomicsWorkbenchR::do_query()
- Target-file sample metadata (defining samples, batches, and experimental design)
- Normalized concentration matrix derived using internal lipid standards

## Outputs

- Batch-corrected normalized concentration matrix
- Differential abundance table for design contrasts (downstream of the corrected matrix)
- MA plot and volcano plot image files reflecting the corrected analysis

## How to apply

In ADViSELipidomics, first load the lipid abundance matrix and target-file sample metadata, then normalize the data matrix using internal lipid standards to obtain absolute concentrations per lipid and sample. In the statistical-analysis tab, the user explicitly opts in to the built-in batch effect correction, which is applied to the normalized concentration matrix before fitting the differential abundance model. The design (simple two-group or complex with covariates) is defined alongside the batch handling, so the corrected matrix is the input to the contrasts evaluated by MA and volcano plots. Note that the article does not name the specific correction algorithm; the missing-signals section states the method name is not given, so the agent should treat the correction as a black-box step of the app and verify outcomes empirically (e.g., replicate QC behavior) rather than assume a known implementation.

## Related tools

- **ADViSELipidomics** (Shiny app that provides the built-in batch effect correction applied to the normalized concentration matrix within the statistical-analysis tab) — https://github.com/ShinyFabio/ADViSELipidomics
- **Shiny** (Framework implementing the ADViSELipidomics interface in which the batch correction option is exposed) — https://github.com/ShinyFabio/ADViSELipidomics
- **LipidSearch** (Source of the lipid identification/quantification output files whose abundance matrix undergoes batch correction in Case Study 1)
- **LIQUID** (Alternative source of lipid identification/quantification outputs accepted by the app for batch-corrected analysis)

## Examples

```
library("ADViSELipidomics"); run_ADViSELipidomics()
```

## Evaluation signals

- The differential abundance analysis runs on the corrected matrix: MA and volcano plots must reflect the model fitted after batch handling is set in the statistical-analysis tab.
- If the experiment includes internal lipid standards, verify the input matrix carries normalized concentration values per lipid and sample before correction is invoked.
- Replicate QC samples (n=2) should show very low variance after correction; note the paper warns the drift plateau seen in 2D pooled QC distributions was not visible in replicate samples, so power to detect non-linear drift is low.
- The exported differential abundance table and plot image files land in the output directory as expected for the chosen design and comparison.
- Behavior should be cross-checked against Case Study 1 LipidSearch example files provided in the ADViSELipidomics_book repository.

## Limitations

- The specific batch correction algorithm is not identified in the paper: 'Batch correction was carried out using' — the method name is not given in the text.
- Power to detect non-linear signal drift is low: the drift pattern visible in pooled QC samples was 'not visible in replicate QC samples because the variance is very low.'
- Correction requires a defined experimental design from the target-file metadata; it cannot be meaningfully applied without sample batch annotations.
- The Shiny interface rendering depends on screen size/resolution, which may make the batch-handling controls hard to locate (adjust zoom with Ctrl/Cmd +/-).

## Evidence

- [abstract] Batch correction is an opt-in step tied to the design definition: "it identifies differentially abundant lipids in simple and complex experimental designs, dealing with batch effect correction."
- [abstract] Normalization precedes correction: "When the experiment includes internal lipid standards, ADViSELipidomics can normalize the data matrix, providing normalized concentration values per lipids and samples."
- [discussion] Batch correction method not named in the paper: [evidence span withheld — anchor status: not_found]
- [results] Correction applied to the normalized concentration matrix before differential analysis: [evidence span withheld — anchor status: not_found]
- [readme] README scope statement covering batch handling: "it allows the identification of differentially abundant lipids in simple and complex experimental designs, dealing with batch effect correction."
