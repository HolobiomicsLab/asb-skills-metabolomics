---
name: metabolomics-data-processing-pipeline
description: Use when you have a preprocessed peak-intensity/concentration matrix
  plus a sample information table (SampleID and group) and need an end-to-end metabolomics
  analysis from normalization to pathway interpretation.
when_to_use_negative:
- Input is raw spectral data requiring peak picking or annotation — iMAP deliberately
  excludes raw data preprocessing and peak picking/annotation modules 'considering
  the risk of data security'
- Analysis goal is purely predictive modeling or network analysis without the three-step
  filtering-selection-interpretation pipeline (though iMAP offers separate modules
  for these)
- Data is not a metabolomics peak/concentration matrix (e.g. genomics reads) outside
  the tool's scope
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0091
tools:
- name: iMAP
  role: Web server implementing the three-step analysis pipeline (data-processing,
    metabolites-selection, data-interpretation)
  repo: https://imap.metaboprofile.cloud/
- name: IP4M
  role: Desktop predecessor platform covering all key steps from pre-processing to
    data-interpretation; iMAP extends it
  repo: https://github.com/IP4M
- name: R-scripts-for-iMAP
  role: Backend R scripts implementing the iMAP modules
  repo: https://github.com/IP4M/R-scripts-for-iMAP
- name: MetaboAnalystR
  role: R package underlying statistical analysis in the metabolites-selection layer
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.3389/fchem.2021.659656
    title: 'iMAP: A Web Server for Metabolomics Data Integrative Analysis'
  source_content_hash: sha256:c0a2ee23321443423c5e39553c71daf2cd7355c90744d68ed4f7064f59c5ad3d
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Hmisc
  - IP4M
  - MetaboAnalystR
  - R
  - R-scripts-for-iMAP
  - iMAP
  - ropls
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 14
---

# metabolomics-data-processing-pipeline

## Summary

A three-step metabolomics data analysis workflow — data-processing, metabolites-selection, and data-interpretation — implemented in the iMAP web server (successor to IP4M), covering data filtering/normalization through statistical metabolite selection to pathway-based interpretation.

## When to use

Use when you have a preprocessed peak-intensity/concentration matrix plus a sample information table (SampleID and group) and need an end-to-end metabolomics analysis from normalization to pathway interpretation.

## When NOT to use

- Input is raw spectral data requiring peak picking or annotation — iMAP deliberately excludes raw data preprocessing and peak picking/annotation modules 'considering the risk of data security'
- Analysis goal is purely predictive modeling or network analysis without the three-step filtering-selection-interpretation pipeline (though iMAP offers separate modules for these)
- Data is not a metabolomics peak/concentration matrix (e.g. genomics reads) outside the tool's scope

## Inputs

- Preprocessed peak-intensity/concentration data matrix (CSV)
- Sample information table with SampleID and group (CSV)
- Optional variable information table (CSV)

## Outputs

- Processed data table after filtering, normalization, batch correction, transformation and scaling (CSV)
- Basic statistics summary and PCA plots
- Selected-metabolites list from univariate/multivariate integration (CSV)
- Pathway analysis results from MSEA and MetPA (CSV)
- Publication-ready figures

## How to apply

Load a preprocessed peak-intensity/concentration data matrix as CSV along with a sample information table (SampleID, group) and an optional variable information table, since iMAP does not implement raw data preprocessing or peak picking/annotation. Run the data-processing module: highlight missing values, remove samples/variables exceeding missing-value thresholds (e.g. >90% zeros in samples and >40% zeros in variables in the demonstration), fill missing values, normalize (e.g. counts per million), correct batch effects, remove QC-unstable variables, then apply transformation and scaling, ending with a statistics summary and PCA. In the metabolites-selection step, apply univariate analysis with user-set P/FDR/|log2FC| thresholds plus multivariate PLS-DA/OPLS-DA VIP-based selection, integrating results via Union/Intersection summary modules. Finally run data-interpretation with MSEA and MetPA pathway analysis modules, and export each layer's intermediate artifact (processed data table, selected-metabolites list, pathway results) in CSV format. Modules from steps 1–3 can be chained into thread-like or tree-like customized workflows.

## Related tools

- **iMAP** (Web server implementing the three-step analysis pipeline (data-processing, metabolites-selection, data-interpretation)) — https://imap.metaboprofile.cloud/
- **IP4M** (Desktop predecessor platform covering all key steps from pre-processing to data-interpretation; iMAP extends it) — https://github.com/IP4M
- **R-scripts-for-iMAP** (Backend R scripts implementing the iMAP modules) — https://github.com/IP4M/R-scripts-for-iMAP
- **MetaboAnalystR** (R package underlying statistical analysis in the metabolites-selection layer)

## Examples

```
Rscript run_imap_layer.R --input peak_intensity.csv --sample_info samples.csv --layer data-processing
```

## Evaluation signals

- Each of the three layers must emit a CSV artifact in iMAP's native output format
- The data-processing layer must terminate with a basic statistics summary and a PCA plot
- Samples/variables removed during filtering must respect the configured missing-value/zero thresholds (demonstration used >90% zeros in samples and >40% zeros in variables)
- Selected metabolites must satisfy the user-set P/FDR/|log2FC| thresholds and/or PLS-DA/OPLS-DA VIP criteria before entering the data-interpretation layer
- PCA of processed data should show batch effects removed and QC-unstable variables eliminated (no QC-sample-driven clustering)

## Limitations

- Modules of raw data preprocessing and peak picking and annotation were not implemented in iMAP 'considering the risk of data security'
- IP4M/iMAP do not fully satisfy demands for highly customized workflows beyond thread-like or tree-like module combinations
- No DOI stated in the discussion section and no version number stated for iMAP or IP4M
- No changelog found for the associated repository

## Evidence

- [intro] three steps summary: "metabolomic data analysis can be summarized as three steps"
- [intro] step definitions: "Step 1: data-processing including data filtering and normalization"
- [methods] processing module chain: "the input data will first go through a data-processing module including highlighting missing value"
- [methods] filtering thresholds: [evidence span withheld — anchor status: not_found]
- [intro] raw data exclusion: "Modules of raw data preprocessing and peak picking and annotation were not implemented considering the risk of data security"
- [methods] backend scripts: "The backend R scripts are available at https://github.com/IP4M/R-scripts-for-iMAP"
- [readme] availability: "iMAP is freely available for academic usage at https://imap.metaboprofile.cloud/ (License MPL 2.0)."
