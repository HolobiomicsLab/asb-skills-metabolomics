---
name: csv-result-artifact-export
description: Use when running or replicating the iMAP three-step workflow (data-processing
  -> metabolites-selection -> data-interpretation) and each layer's output — processed
  data table, selected-metabolites list, pathway results — must be persisted as a
  CSV artifact for downstream reuse or validation.
when_to_use_negative:
- Raw spectral data (e.g. mzML/NetCDF) before preprocessing — iMAP does not implement
  raw data preprocessing or peak picking/annotation modules, so no CSV artifact of
  that kind exists to export
- When the analysis target is a publication-ready figure rather than a reusable tabular
  result — iMAP's upgraded plotting tools serve that purpose instead
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0091
tools:
- name: iMAP
  role: Web platform whose layers produce the CSV artifacts in its native output format
  repo: https://github.com/IP4M
- name: IP4M
  role: Desktop predecessor whose R-based module conventions the CSV exports match
  repo: https://github.com/IP4M
- name: R
  role: Backend language of the iMAP scripts; exports can be reloaded in R for custom
    downstream analysis
  repo: https://github.com/IP4M/R-scripts-for-iMAP
- name: MetaboAnalystR
  role: R package used by iMAP backend modules; can consume exported CSV tables of
    processed data and selected metabolites
  repo: https://github.com/IP4M/R-scripts-for-iMAP
- name: ropls
  role: R package backing PLS-DA/OPLS-DA VIP-based metabolite selection whose VIP
    tables feed the exported selected-metabolites list
  repo: https://github.com/IP4M/R-scripts-for-iMAP
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.3389/fchem.2021.659656
    title: 'iMAP: A Web Server for Metabolomics Data Integrative Analysis'
  source_content_hash: sha256:18f29a350fb64f743f6ea467e108adb4bfebc948fd78413421c2b7d289d01785
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Hmisc
  - IP4M
  - MetaboAnalystR
  - R
  - iMAP
  - ropls
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# csv-result-artifact-export

## Summary

Export the intermediate artifact of each iMAP analysis layer (data-processing, metabolites-selection, data-interpretation) as a CSV file, matching iMAP's native output format so that downstream layers and external tools (R, MetaboAnalystR, ropls) can consume results independently.

## When to use

Use when running or replicating the iMAP three-step workflow (data-processing -> metabolites-selection -> data-interpretation) and each layer's output — processed data table, selected-metabolites list, pathway results — must be persisted as a CSV artifact for downstream reuse or validation.

## When NOT to use

- Raw spectral data (e.g. mzML/NetCDF) before preprocessing — iMAP does not implement raw data preprocessing or peak picking/annotation modules, so no CSV artifact of that kind exists to export
- When the analysis target is a publication-ready figure rather than a reusable tabular result — iMAP's upgraded plotting tools serve that purpose instead

## Inputs

- Preprocessed peak-intensity/concentration data matrix (CSV)
- Sample information table with SampleID and group columns (CSV)
- Optional variable (metabolite) information table (CSV)
- Layer-specific intermediate outputs: processed data table, selected-metabolites list, MSEA/MetPA pathway analysis results

## Outputs

- Processed data table after data-processing (CSV)
- Selected-metabolites list after metabolites-selection (CSV)
- Pathway analysis results (MSEA and MetPA) after data-interpretation (CSV)

## How to apply

After each iMAP layer completes, write its end-product to CSV: (1) the data-processing layer must terminate with a basic statistics summary and PCA, and export the fully filtered/normalized/corrected data table (missing-value handling, >90% zero removal for samples and >40% for variables, normalization, batch correction, QC-based removal of unstable variables, transformation and scaling); (2) the metabolites-selection layer exports the selected-metabolites list that satisfies the user-set P/FDR/|log2FC| thresholds or PLS-DA/OPLS-DA VIP criteria; (3) the data-interpretation layer exports MSEA and MetPA pathway analysis results computed on the selected metabolites. CSV is iMAP's native output format, so exports should match its column conventions (peak-intensity/concentration matrix keyed by SampleID) to remain interoperable with the backend R scripts.

## Related tools

- **iMAP** (Web platform whose layers produce the CSV artifacts in its native output format) — https://github.com/IP4M
- **IP4M** (Desktop predecessor whose R-based module conventions the CSV exports match) — https://github.com/IP4M
- **R** (Backend language of the iMAP scripts; exports can be reloaded in R for custom downstream analysis) — https://github.com/IP4M/R-scripts-for-iMAP
- **MetaboAnalystR** (R package used by iMAP backend modules; can consume exported CSV tables of processed data and selected metabolites) — https://github.com/IP4M/R-scripts-for-iMAP
- **ropls** (R package backing PLS-DA/OPLS-DA VIP-based metabolite selection whose VIP tables feed the exported selected-metabolites list) — https://github.com/IP4M/R-scripts-for-iMAP

## Examples

```
write.csv(processed_data, 'data_processing_output.csv', row.names = FALSE); write.csv(selected_metabolites, 'metabolites_selection_output.csv', row.names = FALSE)
```

## Evaluation signals

- Each of the three layers (data-processing, metabolites-selection, data-interpretation) emits exactly one CSV artifact
- The data-processing CSV is produced only after the layer terminates with a basic statistics summary and PCA
- The selected-metabolites CSV contains only metabolites passing the user-set P/FDR/|log2FC| thresholds or VIP-based selection, as integrated by the Union/Intersection summary modules
- Column layout of each export matches iMAP's native CSV format (SampleID-keyed intensity/concentration matrix) so R scripts can reload it
- The data-interpretation CSV is computed exclusively on the exported selected-metabolites list, not on the full processed table

## Limitations

- iMAP implements neither raw data preprocessing nor peak picking/annotation, so no CSV artifact can be exported for those stages
- As a web server, backend computation and data security considerations constrain which intermediate outputs are exposed; detailed limitations are acknowledged but not specified in the article

## Evidence

- [intro] three-step workflow: "metabolomic data analysis can be summarized as three steps"
- [methods] native CSV output chain: [evidence span withheld — anchor status: not_found]
- [methods] processing layer end state: "the input data will first go through a data-processing module including highlighting missing value"
- [intro] no raw preprocessing: "Modules of raw data preprocessing and peak picking and annotation were not implemented considering the risk of data security"
- [discussion] web tool scope: "This study reported a new web tool, iMAP, for metabolomics data analysis."
