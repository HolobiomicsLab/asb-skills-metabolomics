---
name: metaboset-exprs-matrix-manipulation
description: Reference for when working with preprocessed LC-MS metabolomics data
  inside the notame R package and needing to extract, transform, or export the feature
  abundance matrix held by a MetaboSet object.
when_to_use_negative:
- Input is still raw LC-MS instrument files — notame expects data already peak-picked
  (e.g. via MS-DIAL) into a feature table
- The data are not LC-MS metabolomics feature abundances and have no MetaboSet-compatible
  feature/sample structure
- Only batch effect correction or drift correction is needed without touching the
  exprs matrix for downstream multivariate analysis
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0091
tools:
- name: notame
  role: R package providing the MetaboSet object and the exprs() accessor used to
    extract the abundance matrix, plus pqn_normalization and scaling for preprocessing
  repo: https://github.com/antonvsdata/notame
- name: R
  role: Language in which notame is implemented and in which the exprs matrix is loaded,
    transformed and exported
- name: liquid chromatography–mass spectrometry
  role: Instrument platform that generated the preprocessed feature abundance data
    stored in the MetaboSet object
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.3390/metabo10040135
    title: '“Notame”: Workflow for Non-Targeted LC–MS Metabolic Profiling'
  source_content_hash: sha256:f0081fd2d400b20f148c7cf0fc7941f0f2a95e6603cfba76e458aa921c118e3a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/antonvsdata/notame
  tools:
  - R
  - liquid chromatography–mass spectrometry
  - notame
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# metaboset-exprs-matrix-manipulation

## Summary

Extract and manipulate the feature abundance matrix stored in notame's MetaboSet object via the exprs() accessor, so that downstream transformations (log transform, PQN, autoscaling) and exports operate on a consistent feature-by-sample numeric matrix.

## When to use

Reference for when working with preprocessed LC-MS metabolomics data inside the notame R package and needing to extract, transform, or export the feature abundance matrix held by a MetaboSet object.

## When NOT to use

- Input is still raw LC-MS instrument files — notame expects data already peak-picked (e.g. via MS-DIAL) into a feature table
- The data are not LC-MS metabolomics feature abundances and have no MetaboSet-compatible feature/sample structure
- Only batch effect correction or drift correction is needed without touching the exprs matrix for downstream multivariate analysis

## Inputs

- MetaboSet object from notame holding the feature abundance matrix, feature and sample information
- Preprocessed LC-MS feature abundance matrix saved as full_data.RData
- MS-DIAL Excel feature table (upstream source of the MetaboSet data)

## Outputs

- Feature-by-sample numeric expression matrix (transformed_exprs.csv)
- transformation_parameters.json recording transformation type, PQN reference/method, and center/scale settings

## How to apply

Load the preprocessed MetaboSet abundances (e.g. from full_data.RData) and extract the abundance matrix with exprs() from the notame package; MetaboSet keeps features, sample metadata and the abundance matrix together so functions take one object rather than loose matrices. Confirm the matrix is feature-by-sample with dimensions matching the input data before transforming. In the notame preprocessing workflow the extracted matrix is log-transformed (natural log or glog, since the vignette runs preprocessing on log-transformed data because it follows the model assumptions better), then normalized by probabilistic quotient normalization (pqn_normalization with a chosen reference and method), then autoscaled by mean-centering and dividing by the standard deviation per feature. Finally write the transformed matrix to transformed_exprs.csv and record transformation parameters (transformation type, PQN reference/method, center/scale flags) to transformation_parameters.json.

## Related tools

- **notame** (R package providing the MetaboSet object and the exprs() accessor used to extract the abundance matrix, plus pqn_normalization and scaling for preprocessing) — https://github.com/antonvsdata/notame
- **R** (Language in which notame is implemented and in which the exprs matrix is loaded, transformed and exported)
- **liquid chromatography–mass spectrometry** (Instrument platform that generated the preprocessed feature abundance data stored in the MetaboSet object)

## Examples

```
library(notame); load("full_data.RData"); exprs_matrix <- exprs(metaboset_obj)
```

## Evaluation signals

- transformed_exprs.csv exists and is a feature-by-sample numeric matrix with the same dimensions as the input exprs matrix
- transformation_parameters.json records the transformation type, PQN reference and method, and center/scale settings
- Expert review confirms the transformation and scaling choices match the Section 3.2.4 conventions of the notame paper
- Each function call operates on a single MetaboSet object rather than juggling separate matrices/data frames

## Limitations

- The notame package API is still quite experimental and breaking changes are possible (as stated in the README)
- The original repository (antonvsdata/notame) is migrated and no longer updated; active development is at hanhineva-lab/notame
- notame requires R version 3.5.0 or greater

## Evidence

- [methods] MetaboSet stores features, samples and exprs together: "```mark_nas``` replaces given values in ```exprs``` part of the MetaboSet object with ```NA```."
- [readme] MetaboSet single-interface rationale: [evidence span withheld — anchor status: external_doc]
- [intro] Workflow positioning: "This package can be used to analyze preprocessed LC-MS data in non-targeted metabolomics"
- [readme] Peak picking upstream requirement: [evidence span withheld — anchor status: external_doc]
