---
name: metadata-attribute-column-mapping
description: Use when a user must supply per-sample natural-language metadata for
  metabolomics submissions and needs a template CSV whose columns exactly match the
  selected sample types and attributes before SMetaS standardization.
when_to_use_negative:
- The completed metadata matrix already exists and needs term standardization — that
  is the role of SMetaS's second component, not this matrix-generation step
- Metadata are already controlled-vocabulary terms and need no freetext collection
  template
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3174
tools:
- name: SMetaS
  role: Frontend/API component that captures user selections of sample types and attributes
    and generates the downloadable CSV metadata matrix
  repo: github.com/metabolomics-us/metadatastandardizer
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.3390/metabo13080941
    title: 'SMetaS: A Sample Metadata Standardizer for Metabolomics'
  source_content_hash: sha256:76192d0df00616e971c907167e2e5f5034245fcde5248c3ad1e54e5e29cc475c
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - SMetaS
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# metadata-attribute-column-mapping

## Summary

Generates a downloadable CSV sample-metadata matrix from user-selected core sample types and additional sample attributes in the SMetaS frontend, where each selected sample becomes one row and each selected metadata attribute becomes one column for local completion.

## When to use

Use when a user must supply per-sample natural-language metadata for metabolomics submissions and needs a template CSV whose columns exactly match the selected sample types and attributes before SMetaS standardization.

## When NOT to use

- The completed metadata matrix already exists and needs term standardization — that is the role of SMetaS's second component, not this matrix-generation step
- Metadata are already controlled-vocabulary terms and need no freetext collection template

## Inputs

- User selections of core sample types (frontend, via SMetaS API)
- User selections of additional sample metadata attributes
- List of samples to include (one row per sample)

## Outputs

- sample_metadata_matrix.csv — one row per selected sample, one column per selected metadata attribute, header line of attribute names, empty cells for local completion

## How to apply

Capture the user's frontend selections of core sample types and additional metadata attributes submitted through the SMetaS API. Construct the sample-metadata matrix using the SMetaS frontend/API component so that each selected sample becomes one row and each selected metadata attribute becomes one column. Serialize the matrix to CSV with attribute names as the header line and empty cells, so users can fill cells locally with natural-language terms. The completed file is then reuploaded to the second SMetaS component, which replaces freetext terms with fixed-vocabulary terms via synonym matching and typographical fixing in an n-grams/nearest neighbors model approach.

## Related tools

- **SMetaS** (Frontend/API component that captures user selections of sample types and attributes and generates the downloadable CSV metadata matrix) — github.com/metabolomics-us/metadatastandardizer

## Evaluation signals

- sample_metadata_matrix.csv exists and parses as valid CSV
- CSV header columns exactly match the attributes the user selected in the frontend
- The file contains exactly one row per selected sample
- Data cells are empty, ready to be filled locally with natural-language terms before reupload to the second SMetaS component

## Limitations

- The matrix only structures metadata collection; it does not standardize terms — standardization happens after the completed file is reuploaded to the second component
- The public example instance (metasmetas.us-west-2.elasticbeanstalk.com) is slow because of low i/o speed on the AWS instance, which may affect interactive matrix generation

## Evidence

- [results] Matrix generation from selections: [evidence span withheld — anchor status: not_found]
- [results] Reupload for standardization: [evidence span withheld — anchor status: not_found]
- [abstract] Standardization method: "This transformation process is designed to maximize simplicity and is guided by, among other strategies, synonym matching and typographical fixing in an n-grams/nearest neighbors model approach."
- [readme] Documentation pointer: "Extensive documentation is available at https://metabolomics-us.github.io/metadatastandardizer/"
