---
name: frontend-form-selection-handling
description: Use when a user-facing metabolomics submission form must translate checkbox/selection
  input (core sample types plus additional metadata attributes) into a structured
  template file for local completion.
when_to_use_negative:
- The user has already completed a metadata matrix with natural-language terms — this
  belongs to SMetaS's second (standardization) component, not the matrix-generation
  step.
- Metadata is already structured with fixed-vocabulary terms and does not need template
  generation for local freetext completion.
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_3174
tools:
- name: SMetaS
  role: Provides the frontend and API through which user selections of sample types
    and metadata attributes are captured and converted into the downloadable CSV metadata
    matrix
  repo: github.com/metabolomics-us/metadatastandardizer
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.3390/metabo13080941
    title: 'SMetaS: A Sample Metadata Standardizer for Metabolomics'
  source_content_hash: sha256:bcff816505d077010a6d8dbddd36c15df173f83bc270c86fe9998cb7d5ab9105
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - SMetaS
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# frontend-form-selection-handling

## Summary

Captures user selections of core sample types and additional sample attributes in the SMetaS frontend/API and converts them into a downloadable CSV sample-metadata matrix (one row per sample, one column per attribute) that users fill locally with natural-language terms before reupload for standardization.

## When to use

Use when a user-facing metabolomics submission form must translate checkbox/selection input (core sample types plus additional metadata attributes) into a structured template file for local completion.

## When NOT to use

- The user has already completed a metadata matrix with natural-language terms — this belongs to SMetaS's second (standardization) component, not the matrix-generation step.
- Metadata is already structured with fixed-vocabulary terms and does not need template generation for local freetext completion.

## Inputs

- User frontend selections of core sample types
- User frontend selections of additional sample metadata attributes
- List of selected samples to appear as matrix rows

## Outputs

- sample_metadata_matrix.csv — a downloadable CSV with attribute names as the header line, one row per sample, and empty cells for local completion with natural-language terms

## How to apply

Capture the user's frontend selections of core sample types and additional metadata attributes as they are submitted through the SMetaS API. Construct the sample-metadata matrix so each selected sample becomes one row and each selected metadata attribute becomes one column. Serialize the matrix to CSV, using the attribute names as the header line and leaving cells empty for the user to fill locally with natural-language terms; this completed file is later reuploaded to SMetaS's second component, which replaces freetext terms with fixed-vocabulary terms via synonym matching and typographical fixing in an n-grams/nearest neighbors model approach. The matrix generation should be exposed through the SMetaS frontend/API combination delivered in the containerized environment.

## Related tools

- **SMetaS** (Provides the frontend and API through which user selections of sample types and metadata attributes are captured and converted into the downloadable CSV metadata matrix) — github.com/metabolomics-us/metadatastandardizer

## Evaluation signals

- sample_metadata_matrix.csv exists and parses as valid CSV.
- The CSV header line exactly matches the metadata attributes the user selected in the frontend form.
- The CSV contains exactly one row per selected sample.
- Data cells are empty, ready for users to fill locally with natural-language terms.
- The generated file is downloadable from the SMetaS frontend and acceptable for reupload to the second SMetaS component.

## Limitations

- This step only generates the empty template matrix; it does not standardize terms — term standardization (synonym matching, typographical fixing, n-grams/nearest neighbors model) happens after users complete and reupload the file.
- The hosted example instance is slow because of low i/o speed on the AWS Elastic Beanstalk instance; the project can run much faster on better-provisioned infrastructure.

## Evidence

- [results] Matrix generation from user selections: [evidence span withheld — anchor status: not_found]
- [results] Local completion and reupload flow: [evidence span withheld — anchor status: not_found]
- [abstract] Standardization rationale for the downstream step: [evidence span withheld — anchor status: not_found]
- [abstract] Tool architecture: "SMetaS (Sample Metadata Standardizer) combines a classic database with an API and frontend and is provided in a containerized environment."
- [readme] Deployment caveat: "This instance is slow because we chose to pay for low i/o speed on our AWS instance. The project can run much faster."
