---
name: lc-ms-feature-table-processing
description: Use when you have an Asari feature table (preferred_Feature_table.tsv
  or full_feature_table.tsv) from an LC-MS experiment and need QC curation or grouping
  of features into empirical compounds (empCpds) before MS1/MS2 annotation.
when_to_use_negative:
- The input is still raw mzML/.raw data that has not been preprocessed — run Asari
  preprocessing to obtain a feature table first.
- You already have an empCpd.json or a fully annotated feature table — regrouping
  or re-QC would duplicate completed steps.
- The data are GC-MS or other non-LC-MS types — PCPFM/Asari currently targets LC-MS
  metabolomics and GC support is still under development.
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: pcpfm
  role: 'Pipeline that executes the feature table operations: build_empCpds for pre-annotation,
    plus QC/normalization/imputation operations; installed via pip install pcpfm'
  repo: https://github.com/shuzhao-li-lab/PythonCentricPipelineForMetabolomics
- name: Asari
  role: Produces the input feature tables from raw LC-MS data; its full_feature_table.tsv
    is the recommended input for build_empCpds
- name: khipu
  role: Called by build_empCpds to group co-eluting features (isotopes, adducts) into
    empirical compounds as pre-annotation
- name: MetDataModel
  role: Provides the core data structures (Feature, Empirical compound) that define
    the empCpd JSON-compatible, chainable output format
  repo: https://github.com/shuzhao-li-lab/metDataModel
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1371/journal.pcbi.1011912
    title: Common data models to streamline metabolomics processing and annotation,
      and implementation in a Python pipeline
  source_content_hash: sha256:5eb35855838fa49a7e0f1c3d8d26049fbb9a620c35248f4086782406eb1be1d8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/shuzhao-li-lab/PythonCentricPipelineForMetabolomics
  tools:
  - Asari
  - MetDataModel
  - Python
  - khipu
  - pcpfm
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 18
---

# lc-ms-feature-table-processing

## Summary

Curation of Asari-derived LC-MS feature tables within the pcpfm pipeline: quality-control operations (blank masking, normalization, batch correction, missing value imputation, removal of uncommon features) and pre-annotation grouping of co-eluting adducts/isotopologues into empirical compounds via khipu, yielding empCpd.json for downstream MS1/MS2 annotation.

## When to use

Use when you have an Asari feature table (preferred_Feature_table.tsv or full_feature_table.tsv) from an LC-MS experiment and need QC curation or grouping of features into empirical compounds (empCpds) before MS1/MS2 annotation.

## When NOT to use

- The input is still raw mzML/.raw data that has not been preprocessed — run Asari preprocessing to obtain a feature table first.
- You already have an empCpd.json or a fully annotated feature table — regrouping or re-QC would duplicate completed steps.
- The data are GC-MS or other non-LC-MS types — PCPFM/Asari currently targets LC-MS metabolomics and GC support is still under development.

## Inputs

- Asari feature table (preferred_Feature_table.tsv or asari_results/export/full_feature_table.tsv)
- Raw LC-MS acquisitions (.mzML preferred; .raw with limited support)
- Sample metadata / sequence CSV with sample type (STD/BLANK/etc.), Batch, and file name/path fields
- User-specified empCpd moniker (name for the empirical compound JSON)

## Outputs

- empCpd.json in the experiment's annotations directory: empirical compounds with constituent features and adduct/isotope assignments (pre-annotation)
- Curated feature tables (.tsv) that may be blank masked, normalized, batch corrected, or imputed
- Annotation tables (sample_annotation_table, feature_table, annotation_table) under the results directory
- QA/QC figures (pca.png, tsne.png) per feature table

## How to apply

Load the Asari-derived feature table (the full feature table is recommended) from the experiment's asari_results export directory using the pcpfm pipeline. For pre-annotation, invoke the build_empCpds operation, which calls khipu to grid features into empirical compounds using default adduct lists appropriate to the inferred ionization mode, charges up to z=3, and m+13C3 isotopologues, applying the Orbitrap defaults of 5 ppm m/z tolerance (--khipu_mz_tolerance) and 2 second RT tolerance (--khipu_rt_tolerance) so co-eluting isotopologues and adducts are grouped. Save the result under a user-specified empCpd moniker as empCpd.json in the annotations directory; singleton features without inferred adducts are retained as ungrouped entries, and the pre-annotation prevents degenerate features from being queried separately in later MS1 annotation. For data-quality correction, apply pcpfm's QC operations — normalization, blank masking, batch correction, removal of uncommon features, and missing value imputation — plus default outlier removal dropping samples with an absolute number-of-features Z-score greater than 2.5 (failed injections). The rationale is that the empirical compound is an operational unit for a tentative metabolite that accommodates unresolved isomers and annotation uncertainty, keeping downstream annotation and statistics robust.

## Related tools

- **pcpfm** (Pipeline that executes the feature table operations: build_empCpds for pre-annotation, plus QC/normalization/imputation operations; installed via pip install pcpfm) — https://github.com/shuzhao-li-lab/PythonCentricPipelineForMetabolomics
- **Asari** (Produces the input feature tables from raw LC-MS data; its full_feature_table.tsv is the recommended input for build_empCpds)
- **khipu** (Called by build_empCpds to group co-eluting features (isotopes, adducts) into empirical compounds as pre-annotation)
- **MetDataModel** (Provides the core data structures (Feature, Empirical compound) that define the empCpd JSON-compatible, chainable output format) — https://github.com/shuzhao-li-lab/metDataModel

## Examples

```
pcpfm build_empCpds --khipu_mz_tolerance 5 --khipu_rt_tolerance 2
```

## Evaluation signals

- empCpd.json exists and parses as valid JSON in the experiment's annotations directory.
- Each empirical compound contains its grouped constituent features with assigned adducts/isotopologues; singleton features (without inferred adducts) are retained as ungrouped entries.
- Grouped features co-elute within the RT tolerance (default 2 s) and fall within 5 ppm m/z tolerance of the adduct/isotope expectations.
- Feature tables are in standardized formats (.txt, JSON) immediately usable for downstream analysis in MetaboAnalyst, R, or Python.
- Failed injections are removed: samples with an absolute number-of-features Z-score greater than 2.5 are dropped by default.

## Limitations

- Support for .raw files is limited; mzML (e.g., via ThermoRawFileParser conversion) is the preferred input format.
- GC and other data types are not yet supported by the pipeline.
- Downloaded annotation sources such as HMDB are restricted to non-commercial use and are not redistributed with pcpfm.
- If the experiment does not resolve isomers, an empirical compound can be any one or a mixture of the isomers — pre-annotation is tentative, not identification.

## Evidence

- [readme] pre-annotation to group featues to empirical compounds (khipu): "pre-annotation to group featues to empirical compounds (khipu)"
- [readme] This includes feature tables that are optionally blank masked, normalized, batch corrected, annotated or otherwise curated by PCPFM: "This includes feature tables that are optionally blank masked, normalized, batch corrected, annotated or otherwise curated by PCPFM"
- [readme] An empirical compound (empCpd) is a computational unit for a tentative metabolite, based on the experimental measurement.: [evidence span withheld — anchor status: external_doc]
- [readme] The step of assigning features to empirical compound is called pre-annotation: [evidence span withheld — anchor status: external_doc]
- [intro] Operations to correct common data quality issues are provided including normalization, blank masking, batch correction, removal of uncommon features, and missing value imputation.: "Operations to correct common data quality issues are provided including normalization, blank masking, batch correction, removal of uncommon features, and missing value imputation."
- [intro] by default, samples that have an absolute value for their number of features Z-score greater than 2.5 are dropped.: "by default, samples that have an absolute value for their number of features Z-score greater than 2.5 are dropped."
