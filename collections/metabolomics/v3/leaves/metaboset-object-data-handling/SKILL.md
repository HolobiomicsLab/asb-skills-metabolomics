---
name: metaboset-object-data-handling
description: 'Use when loading or manipulating preprocessed LC-MS metabolomics data
  inside the notame workflow: constructing, reading, or passing a MetaboSet object
  (e.g. full_data.RData) through preprocessing or modeling steps.'
when_to_use_negative:
- Input is raw LC-MS instrument data (e.g. mzML/raw files) that still needs peak picking
  — notame expects an MS-DIAL peak table, not raw spectra
- The analysis goal is a standalone matrix operation outside the notame pipeline where
  a plain data frame or matrix is sufficient
edam_topics:
- http://edamontology.org/topic_3173
tools:
- name: notame
  role: Provides the MetaboSet container class and all functions (mark_nas, correct_drift,
    flag_detection, impute_rf) that read and return MetaboSet objects
  repo: https://github.com/antonvsdata/notame
- name: R
  role: Runtime language in which the MetaboSet object is created, loaded (load/full_data.RData)
    and manipulated
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.3390/metabo10040135
    title: '“Notame”: Workflow for Non-Targeted LC–MS Metabolic Profiling'
  source_content_hash: sha256:2b522d3c2dd6f6f6c27162ec474150774ed62d85d2d48322a0459fcc3edee710
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/antonvsdata/notame
  tools:
  - MUVR
  - R
  - notame
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3173
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# metaboset-object-data-handling

## Summary

Handles notame's MetaboSet custom R object, which bundles the feature abundance matrix (exprs), sample information and feature information for non-targeted LC-MS metabolomics data into one interface that all notame functions consume and return.

## When to use

Use when loading or manipulating preprocessed LC-MS metabolomics data inside the notame workflow: constructing, reading, or passing a MetaboSet object (e.g. full_data.RData) through preprocessing or modeling steps.

## When NOT to use

- Input is raw LC-MS instrument data (e.g. mzML/raw files) that still needs peak picking — notame expects an MS-DIAL peak table, not raw spectra
- The analysis goal is a standalone matrix operation outside the notame pipeline where a plain data frame or matrix is sufficient

## Inputs

- Excel peak table exported from MS-DIAL (features x samples plus sample and feature metadata)
- Preprocessed, imputed notame MetaboSet object saved as RData (e.g. full_data.RData)
- Feature abundance matrix (exprs) with missing values encoded as NA

## Outputs

- MetaboSet object in the R session carrying exprs, sample info and feature info
- Persisted MetaboSet object as .RData for downstream multivariate analyses

## How to apply

Store the data in a MetaboSet object rather than loose matrices/data frames: the object holds the feature abundance matrix (exprs, features x samples), sample metadata and feature metadata, giving all notame functions a single consistent interface. Read MS-DIAL-derived Excel peak tables into a MetaboSet with notame's Excel-reading functionality, and correctly represent missing values first — mark_nas replaces given values in the exprs part of the MetaboSet object with NA before imputation (e.g. random forest imputation via impute_rf). Keep the same object through drift correction (correct_drift), flagging (flag_detection, flag_contaminants, flag_quality), imputation and batch correction, saving the preprocessed, imputed object (e.g. full_data.RData) so downstream supervised analyses like MUVR RF/PLS variable selection can load it directly. Do not strip flags or metadata from the object before modeling, since unflagged features and the class label column are selected from it at the modeling stage.

## Related tools

- **notame** (Provides the MetaboSet container class and all functions (mark_nas, correct_drift, flag_detection, impute_rf) that read and return MetaboSet objects) — https://github.com/antonvsdata/notame
- **R** (Runtime language in which the MetaboSet object is created, loaded (load/full_data.RData) and manipulated)

## Examples

```
library(notame); load("full_data.RData")  # restore preprocessed, imputed MetaboSet object for downstream modeling
```

## Evaluation signals

- The loaded object exposes an exprs abundance matrix with rows as molecular features and columns as samples, plus attached sample and feature metadata
- Missing values in the exprs matrix are represented as R's NA after mark_nas, not as 0 or another placeholder
- Loading full_data.RData in a fresh R session with library(notame) succeeds and the object is accepted by notame functions without re-wrapping
- Downstream MUVR analysis can select unflagged features and the class (group) column directly from the object, confirming flags/metadata were preserved

## Limitations

- The notame package API is described as 'still quite experimental' with breaking changes possible
- The original antonvsdata/notame repository is migrated and no longer maintained; active development is at hanhineva-lab/notame
- Requires R version 3.5.0 or greater, and only the dependencies needed for the intended functions need to be installed (install_dependencies)

## Evidence

- [readme] Data is stored in a custom object that holds all the information about the features and samples along with the feature abundance matrix.: [evidence span withheld — anchor status: external_doc]
- [readme] This allows for a simple interface for all of the functions in the package, as there is no need to juggle with different matrices/data frames.: [evidence span withheld — anchor status: external_doc]
- [methods] mark_nas replaces given values in exprs part of the MetaboSet object with NA.: "```mark_nas``` replaces given values in ```exprs``` part of the ```MetaboSet``` object with ```NA```."
- [intro] This package can be used to analyze preprocessed LC-MS data in non-targeted metabolomics.: "This package can be used to analyze preprocessed LC-MS data in non-targeted metabolomics."
