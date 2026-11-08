---
name: tfidf-nearest-neighbor-model-retraining
description: Use when a user confirms a new freetext sample metadata term that is
  absent from an SMetaS header's existing vocabulary and the standardizer's synonym/typo
  models must be updated to map it.
when_to_use_negative:
- The requested term already exists in the header's vocabulary — no expansion or retraining
  is needed
- Standardization targets a fixed-vocabulary field where the transformation is not
  guided by the n-grams/nearest-neighbors model (e.g., controlled enums without freetext)
edam_topics:
- http://edamontology.org/topic_3336
tools:
- name: SMetaS
  role: Provides the vocabulary files, tf-idf/n-grams pipeline, and nearest-neighbors
    models that are extended and retrained; its containerized frontend consumes the
    updated artifacts
  repo: github.com/metabolomics-us/metadatastandardizer
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.3390/metabo13080941
    title: 'SMetaS: A Sample Metadata Standardizer for Metabolomics'
  source_content_hash: sha256:7d4b5d4bf828a459f8710105fd4f7eb2ab0c7d895e64de16fa08ee085e410659
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - SMetaS
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3336
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# tfidf-nearest-neighbor-model-retraining

## Summary

Extend the SMetaS metadata standardizer's fixed vocabularies with user-confirmed freetext terms (e.g., 'allergen exposure', 'ozone', 'hours/day') and retrain the tf-idf term-document matrix and nearest-neighbors index so future inputs resolve to the new entries.

## When to use

Use when a user confirms a new freetext sample metadata term that is absent from an SMetaS header's existing vocabulary and the standardizer's synonym/typo models must be updated to map it.

## When NOT to use

- The requested term already exists in the header's vocabulary — no expansion or retraining is needed
- Standardization targets a fixed-vocabulary field where the transformation is not guided by the n-grams/nearest-neighbors model (e.g., controlled enums without freetext)

## Inputs

- Existing SMetaS vocabulary file for the target header field
- User-confirmed new freetext terms not present in the vocabulary (e.g., 'allergen exposure', 'ozone', 'hours/day')
- Sample freetext inputs for smoke-test mapping

## Outputs

- Expanded vocabulary file with new selectable target entries
- Retrained tf-idf term-document matrix over all vocabulary entries
- Retrained nearest-neighbors index for synonym matching and typographical fixing
- Smoke-test mapping results showing inputs resolve to the new terms

## How to apply

Load the existing SMetaS vocabulary file for the relevant header and append the confirmed freetext terms as selectable target entries. Re-tokenize the expanded vocabulary into n-grams and rebuild the tf-idf term-document matrix over all vocabulary entries, mirroring SMetaS's n-grams/nearest-neighbors approach for synonym matching and typographical fixing. Retrain the nearest-neighbors index on the refreshed tf-idf vectors. Persist the expanded vocabulary and retrained model files for use by the containerized SMetaS frontend. Validate qualitatively: run smoke-test freetext inputs (e.g., 'allergen exposure', 'ozone', 'hours/day') and confirm they resolve to the newly added entries rather than unrelated vocabulary selections, since the paper reports no quantitative acceptance threshold.

## Related tools

- **SMetaS** (Provides the vocabulary files, tf-idf/n-grams pipeline, and nearest-neighbors models that are extended and retrained; its containerized frontend consumes the updated artifacts) — github.com/metabolomics-us/metadatastandardizer

## Evaluation signals

- Retrained model files and expanded vocabulary persist and are loadable by the containerized SMetaS frontend
- Smoke-test freetext inputs 'allergen exposure', 'ozone', and 'hours/day' map to the newly added vocabulary selections
- Pre-existing vocabulary terms still resolve to their original selections (no regression in nearest-neighbor mappings)
- Rebuilt tf-idf matrix covers all vocabulary entries including the appended n-grams
- No quantitative threshold is reported in the paper, so acceptance is this qualitative mapping check

## Limitations

- No quantitative accuracy threshold for retraining acceptance is reported in the paper; validation is only a qualitative smoke-test mapping check
- Only user-confirmed terms should be appended — unconfirmed freetext suggestions do not trigger vocabulary expansion
- The README notes the example hosted instance is slow due to low i/o speed on the chosen AWS instance, so runtime performance of the deployed frontend may vary

## Evidence

- [abstract] SMetaS replaces freetext sample metadata terms with terms from fixed vocabularies using synonym matching and typographical fixing: [evidence span withheld — anchor status: not_found]
- [abstract] n-grams/nearest neighbors model approach: "This transformation process is designed to maximize simplicity and is guided by, among other strategies, synonym matching and typographical fixing in an n-grams/nearest neighbors model approach."
- [abstract] containerized environment: "SMetaS (Sample Metadata Standardizer) combines a classic database with an API and frontend and is provided in a containerized environment."
- [readme] SMetaS documentation: "Extensive documentation is available at https://metabolomics-us.github.io/metadatastandardizer/"
