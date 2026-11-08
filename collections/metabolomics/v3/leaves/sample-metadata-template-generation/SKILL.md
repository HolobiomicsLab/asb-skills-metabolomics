---
name: sample-metadata-template-generation
description: Use when a metabolomics submission workflow needs a blank sample-metadata
  template (one row per sample, one column per selected attribute) for users to fill
  with freetext terms prior to vocabulary standardization.
when_to_use_negative:
- The user already has a completed sample-metadata CSV — route it to SMetaS's second
  component for vocabulary standardization instead of generating a new template.
- The goal is to standardize freetext metadata into fixed-vocabulary terms — that
  is performed by SMetaS's transformation (n-grams/nearest neighbors synonym matching),
  not by this template-generation step.
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_3277
- http://edamontology.org/topic_0219
tools:
- name: SMetaS
  role: Frontend/API component that accepts user selections of sample types and attributes
    and generates the downloadable CSV metadata matrix
  repo: github.com/metabolomics-us/metadatastandardizer
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.3390/metabo13080941
    title: 'SMetaS: A Sample Metadata Standardizer for Metabolomics'
  source_content_hash: sha256:4d3080af6510e8b38095a86036da669fb4ad451ea53e3c282faebfa149c2c7bb
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - SMetaS
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_3277
  - http://edamontology.org/topic_0219
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# sample-metadata-template-generation

## Summary

Generates a downloadable CSV sample-metadata matrix from a user's selections of core sample types and additional metadata attributes via the SMetaS frontend/API, providing an empty one-row-per-sample template that users complete locally with natural-language terms before resubmission to the standardization component.

## When to use

Use when a metabolomics submission workflow needs a blank sample-metadata template (one row per sample, one column per selected attribute) for users to fill with freetext terms prior to vocabulary standardization.

## When NOT to use

- The user already has a completed sample-metadata CSV — route it to SMetaS's second component for vocabulary standardization instead of generating a new template.
- The goal is to standardize freetext metadata into fixed-vocabulary terms — that is performed by SMetaS's transformation (n-grams/nearest neighbors synonym matching), not by this template-generation step.

## Inputs

- User selections of core sample types (from SMetaS frontend)
- User selections of additional sample metadata attributes (from SMetaS frontend)
- List of samples to include as matrix rows

## Outputs

- sample_metadata_matrix.csv: CSV file with one row per sample, one column per selected metadata attribute, attribute names as the header line, and empty cells for local completion

## How to apply

Capture the user's frontend selections of core sample types and additional metadata attributes as submitted through the SMetaS API. Construct a sample-metadata matrix in which each selected sample becomes one row and each selected metadata attribute becomes one column, using the SMetaS frontend/API component (deployed from the containerized metabolomics-us/metadatastandardizer project). Serialize the matrix to CSV with attribute names as the header line and empty cells, so users can fill cells locally with natural-language terms and reupload the completed file to SMetaS's second component for standardization via synonym matching and typographical fixing in an n-grams/nearest neighbors model approach. This first-component step is deliberately simple: it only scaffolds the metadata collection; the transformation to fixed vocabulary terms happens downstream.

## Related tools

- **SMetaS** (Frontend/API component that accepts user selections of sample types and attributes and generates the downloadable CSV metadata matrix) — github.com/metabolomics-us/metadatastandardizer

## Evaluation signals

- sample_metadata_matrix.csv exists and parses as valid CSV
- The CSV header columns exactly match the attributes the user selected
- The matrix contains exactly one row per selected sample
- Data cells are empty, awaiting local completion with natural-language terms before reupload to the second component

## Limitations

- The SMetaS example hosted instance is slow because low i/o speed was chosen on the AWS Elastic Beanstalk deployment; performance is instance-dependent.
- This step only produces an empty template — it does not itself standardize or validate the freetext metadata terms users enter.

## Evidence

- [results] CSV template generation workflow: [evidence span withheld — anchor status: not_found]
- [abstract] SMetaS architecture: "SMetaS (Sample Metadata Standardizer) combines a classic database with an API and frontend and is provided in a containerized environment."
- [abstract] Standardization method context: "This transformation process is designed to maximize simplicity and is guided by, among other strategies, synonym matching and typographical fixing in an n-grams/nearest neighbors model approach."
- [readme] Deployment caveat: "This instance is slow because we chose to pay for low i/o speed on our AWS instance. The project can run much faster."
