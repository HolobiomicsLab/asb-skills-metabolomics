---
name: csv-tabular-matrix-construction
description: Use when a user needs a blank, sample-by-attribute CSV metadata template
  generated from their selections of core sample types and additional sample attributes,
  for local completion and later standardization with SMetaS.
when_to_use_negative:
- The user already has a completed sample-metadata CSV and needs term standardization
  (that is the second SMetaS component's job, not template generation)
- The metadata is already expressed in fixed-vocabulary terms and no freetext-to-ontology
  mapping is needed
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3172
tools:
- name: SMetaS
  role: Frontend/API component that captures user selections of sample types and metadata
    attributes and generates the downloadable CSV metadata matrix
  repo: github.com/metabolomics-us/metadatastandardizer
- name: MetaS
  role: Documented name of the metadata standardizer web tool that hosts the matrix-generation
    frontend
  repo: https://metabolomics-us.github.io/metadatastandardizer/
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.3390/metabo13080941
    title: 'SMetaS: A Sample Metadata Standardizer for Metabolomics'
  source_content_hash: sha256:adc8e9742660346d5b6fcd8bbfdfa09ec358450eab62ad751552a3e732a63a8d
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MetaS
  - SMetaS
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# csv-tabular-matrix-construction

## Summary

Constructs the SMetaS sample-metadata matrix: a CSV file with one row per selected sample and one column per user-selected metadata attribute, with empty cells for users to fill locally with natural-language terms before resubmission.

## When to use

Use when a user needs a blank, sample-by-attribute CSV metadata template generated from their selections of core sample types and additional sample attributes, for local completion and later standardization with SMetaS.

## When NOT to use

- The user already has a completed sample-metadata CSV and needs term standardization (that is the second SMetaS component's job, not template generation)
- The metadata is already expressed in fixed-vocabulary terms and no freetext-to-ontology mapping is needed

## Inputs

- User selections of core sample types and additional sample attributes (frontend/API request)
- List of samples to be annotated

## Outputs

- sample_metadata_matrix.csv: CSV with attribute names as the header line, one row per sample, empty cells for local completion with natural-language terms

## How to apply

Capture the user's frontend selections of core sample types and additional metadata attributes submitted through the SMetaS API. Build a matrix in which each selected sample becomes one row and each selected metadata attribute becomes one column, using the SMetaS frontend/API component. Serialize the matrix to CSV with attribute names as the header line and empty cells for users to fill locally with natural-language terms. The completed file is then reuploaded to the second SMetaS component, which replaces freetext terms with terms from fixed vocabularies via synonym matching and typographical fixing in an n-grams/nearest neighbors approach.

## Related tools

- **SMetaS** (Frontend/API component that captures user selections of sample types and metadata attributes and generates the downloadable CSV metadata matrix) — github.com/metabolomics-us/metadatastandardizer
- **MetaS** (Documented name of the metadata standardizer web tool that hosts the matrix-generation frontend) — https://metabolomics-us.github.io/metadatastandardizer/

## Evaluation signals

- sample_metadata_matrix.csv exists and parses as valid CSV
- Header line columns exactly match the attributes the user selected
- The file contains exactly one row per selected sample
- Cells are empty, ready for natural-language terms to be filled locally
- Completed reupload is accepted by the second SMetaS component for vocabulary standardization

## Limitations

- The SMetaS AWS-hosted example instance is slow because low i/o speed was chosen; performance depends on deployment
- The paper's discussion notes no changelog was found, so behavior across tool versions may not be traceable

## Evidence

- [results] users select core sample types and additional sample attributes, which generate a downloadable csv metadata matrix: [evidence span withheld — anchor status: not_found]
- [abstract] SMetaS combines a classic database with an API and frontend and is provided in a containerized environment.: "SMetaS (Sample Metadata Standardizer) combines a classic database with an API and frontend and is provided in a containerized environment."
- [readme] Extensive documentation is available at https://metabolomics-us.github.io/metadatastandardizer/: "Extensive documentation is available at https://metabolomics-us.github.io/metadatastandardizer/"
