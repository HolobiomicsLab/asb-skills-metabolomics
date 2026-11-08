---
name: lipidmaps-lmsd-mass-matching
description: Use when a MultiABLER-processed LC-MS/MS feature table needs feature
  annotation, and lipid versus metabolite classification must be assigned by m/z matching
  against HMDB and LipidMaps reference mass databases.
when_to_use_negative:
- Input is raw vendor data rather than the processed MultiABLER feature table — the
  annotation step operates on features already extracted upstream.
- The goal is proteomic data analysis — the article states MultiABLER currently does
  not provide functions to analyze proteomic data.
- Feature table already contains annotation labels and only downstream statistics
  (differential expression, sPLS, rCCA) are required.
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0153
tools:
- name: MultiABLER
  role: R pipeline that supplies the annotation workflow context, the hmdb.lipids.massdb
    reference dataset, and the upstream feature table the annotation is applied to.
  repo: https://github.com/holab-hku/MultiABLER
- name: metid
  role: Annotation engine used to match feature m/z values against HMDB V5.0 and LipidMaps
    LMSD reference entries and assign lipid/metabolite labels.
- name: HMDB
  role: Reference mass database (HMDB V5.0, loaded via hmdb.lipids.massdb) used as
    the metabolite annotation target.
- name: R
  role: Runtime environment in which MultiABLER and metid annotation are executed.
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.isci.2023.106881
    title: Integrative processing of untargeted metabolomic and lipidomic data using
      MultiABLER
  source_content_hash: sha256:d331909daf283e6fd6ac0df179ff175eb4faba09053d4ddecaaa1f060d260ee2
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
  - metid
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0153
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# lipidmaps-lmsd-mass-matching

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Annotates LC-MS/MS features by matching m/z values against HMDB V5.0 and LipidMaps LMSD reference mass databases via the metid package, then labeling LipidMaps matches as lipids and remaining matches as metabolites within the MultiABLER pipeline.

## When to use

Use when a MultiABLER-processed LC-MS/MS feature table needs feature annotation, and lipid versus metabolite classification must be assigned by m/z matching against HMDB and LipidMaps reference mass databases.

## When NOT to use

- Input is raw vendor data rather than the processed MultiABLER feature table — the annotation step operates on features already extracted upstream.
- The goal is proteomic data analysis — the article states MultiABLER currently does not provide functions to analyze proteomic data.
- Feature table already contains annotation labels and only downstream statistics (differential expression, sPLS, rCCA) are required.

## Inputs

- Processed LC-MS/MS feature table (m/z values per feature) from the MultiABLER pipeline
- HMDB V5.0 reference mass database (hmdb.lipids.massdb from MultiABLER)

## Outputs

- Annotated feature table (data frame) with lipid/metabolite labels per feature

## How to apply

Load the HMDB V5.0 reference mass database (hmdb.lipids.massdb from MultiABLER) into R as the metabolite/lipid reference set. Take the feature table produced upstream by the MultiABLER extraction workflow and match each feature's m/z value against HMDB V5.0 entries and LipidMaps (LMSD) entries using metid's annotation engine with an appropriate mass tolerance. Decide the label per feature: features whose m/z overlaps LMSD (LipidMaps) entries are annotated as lipids, while all remaining matched features are labeled as metabolites. Assemble and return the annotated feature table as a data frame containing the annotation label column, ready for downstream joint analysis in MultiABLER.

## Related tools

- **MultiABLER** (R pipeline that supplies the annotation workflow context, the hmdb.lipids.massdb reference dataset, and the upstream feature table the annotation is applied to.) — https://github.com/holab-hku/MultiABLER
- **metid** (Annotation engine used to match feature m/z values against HMDB V5.0 and LipidMaps LMSD reference entries and assign lipid/metabolite labels.)
- **HMDB** (Reference mass database (HMDB V5.0, loaded via hmdb.lipids.massdb) used as the metabolite annotation target.)
- **R** (Runtime environment in which MultiABLER and metid annotation are executed.)

## Examples

```
devtools::install_github("holab-hku/MultiABLER", dependencies = TRUE)
```

## Evaluation signals

- The annotation wrapper executes without error and returns a data frame.
- The returned data frame contains an annotation label column distinguishing lipids from metabolites.
- Features matched to LipidMaps (LMSD) entries are labeled lipids; remaining matched features are labeled metabolites.
- Every input feature m/z is represented in the output annotated table (no dropped rows).
- Expert review confirms the returned table structure matches the expected annotated feature-table schema.

## Limitations

- MultiABLER does not implement other normalization/imputation methods such as kNN imputation, mean normalization, or quantile normalization — annotation labels must feed into MultiABLER's supported downstream analyses.
- MultiABLER currently does not provide functions to analyze proteomic data; annotation applies to metabolomic and lipidomic LC-MS/MS features only.
- Annotation accuracy depends on the mass tolerance used for matching against HMDB V5.0 and LMSD; unmatched features remain unlabeled.

## Evidence

- [results] the metabolomic and lipidomic peaks are annotated using metID against HMDB and LipidMaps, with LipidMaps entries labeled as lipids and the remaining features labeled as metabolites: [evidence span withheld — anchor status: not_found]
