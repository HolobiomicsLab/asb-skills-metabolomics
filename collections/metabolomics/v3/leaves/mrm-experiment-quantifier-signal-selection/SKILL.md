---
name: mrm-experiment-quantifier-signal-selection
description: Use when a processed MRMhubExperiment exists and downstream INTEGRATOR/QUANT
  workflows need one quantifier value per feature per analysis in the MRMhub long
  exchange format.
when_to_use_negative:
- The input is raw instrument data or INTEGRATOR peak-integration output that has
  not yet been drift/batch-corrected and quantified in the mrmhub R package
- The goal is importing an exchange file back into QUANT via import_data_mrmhub()
  rather than producing one
- Downstream consumers accept wide-format tables instead of the canonical long exchange
  layout
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: MRMhub
  role: QUANT (the mrmhub R package) provides the programmatic library whose MRMhubExperiment
    container and post-processing functions (normalize_by_istd, drift/batch correction)
    produce the processed dataset from which the quantifier signal is selected
  repo: https://github.com/SLINGhub/MRMhub
- name: mrmhub
  role: R package implementing QUANT; its MRMhubExperiment object holds the corrected,
    quantified feature intensities/concentrations that are reshaped into the long
    exchange file
  repo: https://github.com/SLINGhub/MRMhub
- name: INTEGRATOR
  role: Consumes the exported long-format CSV exchange file downstream; import_data_mrmhub()
    ingests the emitted quant_exchange_long.csv format
  repo: https://github.com/SLINGhub/MRMhub
- name: MRMkit
  role: The earlier open-source package for automated processing of large-scale targeted
    MS-based metabolomics data that underpins MRMhub's role in producing quantification
    results after LC-MS analysis
  repo: https://github.com/SLINGhub/MRMhub
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1021/acs.analchem.0c03060
    title: 'MRMkit: Automated Data Processing for Large-Scale Targeted Metabolomics
      Analysis'
  source_content_hash: sha256:1da401cb68d0e65bb732db764dc293ea797832084ce9a8fde91c580012cac250
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/SLINGhub/MRMhub
  tools:
  - INTEGRATOR
  - MRMhub
  - MRMkit
  - R
  - mrmhub
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 12
---

# mrm-experiment-quantifier-signal-selection

## Summary

Selects a single quantifier signal per feature from a processed MRMhubExperiment (drift/batch-corrected, quantified) and reshapes the selected intensities/concentrations into the long-format CSV exchange file consumed by MRMhub INTEGRATOR/QUANT downstream workflows.

## When to use

Use when a processed MRMhubExperiment exists and downstream INTEGRATOR/QUANT workflows need one quantifier value per feature per analysis in the MRMhub long exchange format.

## When NOT to use

- The input is raw instrument data or INTEGRATOR peak-integration output that has not yet been drift/batch-corrected and quantified in the mrmhub R package
- The goal is importing an exchange file back into QUANT via import_data_mrmhub() rather than producing one
- Downstream consumers accept wide-format tables instead of the canonical long exchange layout

## Inputs

- Processed MRMhubExperiment (drift/batch-corrected, quantified dataset with feature annotations)
- Feature annotations: feature_id, analyte_id, istd_feature_id, is_quantifier flag
- Analysis metadata: analysis_id, sample_id, qc_type, batch_id

## Outputs

- quant_exchange_long.csv — MRMhub/INTEGRATOR long-format CSV, one row per analysis_id × feature_id with columns analysis_id, feature_id, and the signal column (area)

## How to apply

Load the processed MRMhubExperiment produced by upstream drift/batch correction and quantification, then for each feature_id select the quantifier signal flagged by is_quantifier so each feature_id holds exactly one value per analysis_id, linking features measuring the same compound via istd_feature_id and analyte_id. Reshape the chosen feature variable (feature_intensity or feature_conc) from the dataset table into long format, one row per analysis_id × feature_id, preserving the distinction between missing and zero values. Join analysis-level fields (analysis_id, sample_id, qc_type, batch_id) and feature-level fields (feature_id, analyte_id, istd_feature_id) onto each row to assemble the exchange columns analysis_id, feature_id, and the signal column (area). Write the result as quant_exchange_long.csv matching the MRMhub/INTEGRATOR long layout (tab/comma-separated, one row per analysis × feature). The rationale: INTEGRATOR/QUANT downstream import via import_data_mrmhub() requires this canonical long interface, and the quantifier flag resolves multi-transition features to one representative signal.

## Related tools

- **MRMhub** (QUANT (the mrmhub R package) provides the programmatic library whose MRMhubExperiment container and post-processing functions (normalize_by_istd, drift/batch correction) produce the processed dataset from which the quantifier signal is selected) — https://github.com/SLINGhub/MRMhub
- **mrmhub** (R package implementing QUANT; its MRMhubExperiment object holds the corrected, quantified feature intensities/concentrations that are reshaped into the long exchange file) — https://github.com/SLINGhub/MRMhub
- **INTEGRATOR** (Consumes the exported long-format CSV exchange file downstream; import_data_mrmhub() ingests the emitted quant_exchange_long.csv format) — https://github.com/SLINGhub/MRMhub
- **MRMkit** (The earlier open-source package for automated processing of large-scale targeted MS-based metabolomics data that underpins MRMhub's role in producing quantification results after LC-MS analysis) — https://github.com/SLINGhub/MRMhub

## Examples

```
mexp <- MRMhubExperiment() |> import_data_mrmhub(path = demo_file, import_metadata = TRUE) |> normalize_by_istd(); save_report_xlsx(mexp, path = "results.xlsx")
```

## Evaluation signals

- No feature_id appears more than once per analysis_id in the emitted table
- Row count equals the number of analyses × the number of features (features in annot_features)
- The distinction between missing and zero signal values is preserved in the long CSV
- The CSV matches the MRMhub/INTEGRATOR long layout (analysis_id, feature_id, area) and can be ingested by import_data_mrmhub()
- Every row carries both analysis-level fields (analysis_id, sample_id, qc_type, batch_id) and feature-level fields (feature_id, analyte_id, istd_feature_id)

## Limitations

- QUANT is implemented in R rather than Python, so the skill assumes an R/mrmhub environment
- The quantifier selection relies on the is_quantifier annotation being correct for each feature; misannotation propagates into the exchange file
- Only the selected feature variable (feature_intensity or feature_conc) is exported; other per-feature variables are dropped
- Exchange file is the canonical MRMhub/INTEGRATOR long layout; other consumers expecting wide tables will not parse it directly

## Evidence

- [results] QUANT selects one quantifier signal per feature and emits the long-format CSV exchange file consumed by INTEGRATOR/QUANT; INTEGRATOR import via import_data_mrmhub() ingests this format.: [evidence span withheld — anchor status: not_found]
- [methods] QUANT is implemented in R rather than Python because the bulk of its work is statistical and visual: "QUANT is implemented in R rather than Python because the bulk of its work is statistical and visual"
- [methods] MRMhub / INTEGRATOR *(recommended)* | [`import_data_mrmhub()`](../reference/import_data_mrmhub.html): "MRMhub / INTEGRATOR *(recommended)* | [`import_data_mrmhub()`](../reference/import_data_mrmhub.html)"
- [readme] INTEGRATOR... reads mzML... and exports integrated peak areas that feed directly into QUANT or other post-processing tools.: [evidence span withheld — anchor status: external_doc]
- [readme] QUANT is a programmatic library for building tailored, reproducible post-processing and quality-control pipelines. It reads INTEGRATOR results directly, or feature-intensity data from other sources (CSV, mzTab-M, Skyline).: [evidence span withheld — anchor status: not_found]
