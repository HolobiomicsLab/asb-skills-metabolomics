---
name: lcms-feature-annotation-against-hmdb
description: Use when a MultiABLER-generated feature table of metabolomic or lipidomic
  peaks needs compound-level annotation and lipid-versus-metabolite labels before
  downstream statistical analysis such as sPLS or rCCA.
when_to_use_negative:
- Input is not yet a processed feature table — MultiABLER's upstream peak-detection
  and feature-table-generation steps must run first
- The analysis involves proteomic data, which MultiABLER does not currently support
  (proteomic integration is planned for future development)
- The annotation target is not HMDB/LipidMaps — e.g. spectra-level compound identification
  requiring MS/MS spectral matching is outside this skill's scope
edam_operation: http://edamontology.org/operation_3648
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0153
tools:
- name: MultiABLER
  role: R pipeline providing the metid_annotate wrapper, the HMDB V5.0 reference mass
    database (hmdb.lipids.massdb), and the feature table that is annotated
  repo: https://github.com/holab-hku/MultiABLER
- name: metid
  role: Annotation engine used within MultiABLER to match feature m/z values against
    HMDB V5.0 and LMSD entries and assign lipid/metabolite labels
- name: R
  role: Runtime environment in which the MultiABLER annotation wrapper and metid package
    are executed
- name: devtools
  role: Used to install MultiABLER from GitHub before running the annotation step
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.isci.2023.106881
    title: Integrative processing of untargeted metabolomic and lipidomic data using
      MultiABLER
  source_content_hash: sha256:22a0f58146bc6370060e5877c45a73690f7193a8805cedf8af58b912b4e0c6be
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/holab-hku/MultiABLER
  repo_url: https://github.com/holab-hku/MultiABLER
  tools:
  - HMDB
  - MultiABLER
  - R
  - devtools
  - metid
  edam_operation: http://edamontology.org/operation_3648
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0153
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 14
---

# lcms-feature-annotation-against-hmdb

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Annotates processed LC-MS/MS features from the MultiABLER pipeline against the HMDB V5.0 reference mass database (hmdb.lipids.massdb) using metID, labeling LMSD/LipidMaps matches as lipids and remaining matched features as metabolites.

## When to use

Use when a MultiABLER-generated feature table of metabolomic or lipidomic peaks needs compound-level annotation and lipid-versus-metabolite labels before downstream statistical analysis such as sPLS or rCCA.

## When NOT to use

- Input is not yet a processed feature table — MultiABLER's upstream peak-detection and feature-table-generation steps must run first
- The analysis involves proteomic data, which MultiABLER does not currently support (proteomic integration is planned for future development)
- The annotation target is not HMDB/LipidMaps — e.g. spectra-level compound identification requiring MS/MS spectral matching is outside this skill's scope

## Inputs

- Processed LC-MS/MS feature table from the MultiABLER pipeline (m/z values and peak intensities per feature)
- HMDB V5.0 reference mass database (hmdb.lipids.massdb) included with MultiABLER
- LMSD (LipidMaps) entries used for lipid classification

## Outputs

- Annotated feature table as an R data frame with an annotation label column distinguishing lipids (LMSD/LipidMaps matches) from metabolites (other HMDB matches)

## How to apply

Load the HMDB V5.0 reference mass database (hmdb.lipids.massdb) into R as the metabolite/lipid reference set, then read the processed LC-MS/MS feature table produced by the MultiABLER workflow. Match each feature's m/z value against HMDB V5.0 entries and against LMSD (LipidMaps) entries using the metid package's annotation engine with an appropriate mass tolerance. Decision point: features whose m/z overlaps LMSD entries are labeled as lipids, while all other matched features are labeled as metabolites. Assemble and return the annotated feature table as an R data frame containing the annotation label column, which can then feed the statistical analyses the paper validates (differential expression analysis, sPLS, and rCCA). Correctness is judged by the wrapper executing without error and returning a data frame with the expected annotation label column.

## Related tools

- **MultiABLER** (R pipeline providing the metid_annotate wrapper, the HMDB V5.0 reference mass database (hmdb.lipids.massdb), and the feature table that is annotated) — https://github.com/holab-hku/MultiABLER
- **metid** (Annotation engine used within MultiABLER to match feature m/z values against HMDB V5.0 and LMSD entries and assign lipid/metabolite labels)
- **R** (Runtime environment in which the MultiABLER annotation wrapper and metid package are executed)
- **devtools** (Used to install MultiABLER from GitHub before running the annotation step)

## Examples

```
devtools::install_github("holab-hku/MultiABLER", dependencies = TRUE)
```

## Evaluation signals

- The annotation wrapper executes without error on the processed feature table
- The returned object is an R data frame containing the annotation label column
- Features matching LMSD (LipidMaps) entries carry the lipid label, while remaining matched features carry the metabolite label
- Annotated m/z values are consistent with entries in the HMDB V5.0 reference database within the applied mass tolerance
- The labeled feature table is suitable to feed downstream analyses validated in the paper (differential expression analysis, sPLS, rCCA)

## Limitations

- MultiABLER does not provide functions to analyze proteomic data, which is another important output of the MultiABLER extraction method; proteomic integration is planned for future development
- Other data manipulation and normalization methods such as kNN imputation, mean normalization, and quantile normalization are not implemented in MultiABLER, so features requiring those treatments must be handled outside this skill
- Annotation relies on m/z matching against HMDB V5.0 and LMSD entries; features without a database match will remain unannotated

## Evidence

- [other] metabolomic and lipidomic peaks are annotated using metID against HMDB and LipidMaps, with LipidMaps entries labeled as lipids: [evidence span withheld — anchor status: not_found]
