---
name: metadata-vocabulary-extension
description: Use when a user has confirmed a freetext sample-metadata term that is
  absent from an existing SMetaS vocabulary and the tf-idf/nearest-neighbors matching
  models must be rebuilt to recognize it.
when_to_use_negative:
- The freetext terms already exist in the target vocabulary — use SMetaS's normal
  synonym-matching/typo-fixing standardization instead of extending it
- New terms are not user-confirmed and would only be guessed additions to the vocabulary
- The task is bulk standardization of an existing metadata table rather than maintenance
  of the vocabulary/models themselves
tools:
- name: SMetaS
  role: Provides the vocabulary files, tf-idf/nearest-neighbors synonym-matching models,
    and containerized frontend/API whose models are extended and retrained
  repo: github.com/metabolomics-us/metadatastandardizer
- name: MetaS
  role: Hosts the extensive documentation consulted for running and maintaining the
    metadata standardizer instance
  repo: https://metabolomics-us.github.io/metadatastandardizer/
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.3390/metabo13080941
    title: 'SMetaS: A Sample Metadata Standardizer for Metabolomics'
  source_content_hash: sha256:1b3cb5f0381c41a09d14c27a7e027e351d298fc8dd0d05f2cb5096d79b4c7a9d
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MetaS
  - SMetaS
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# metadata-vocabulary-extension

## Summary

Extends SMetaS sample-metadata vocabularies with user-confirmed freetext terms (e.g., 'allergen exposure', 'ozone', 'hours/day') and retracts/retrains the n-grams/tf-idf/nearest-neighbors models so future mappings resolve to the new entries.

## When to use

Use when a user has confirmed a freetext sample-metadata term that is absent from an existing SMetaS vocabulary and the tf-idf/nearest-neighbors matching models must be rebuilt to recognize it.

## When NOT to use

- The freetext terms already exist in the target vocabulary — use SMetaS's normal synonym-matching/typo-fixing standardization instead of extending it
- New terms are not user-confirmed and would only be guessed additions to the vocabulary
- The task is bulk standardization of an existing metadata table rather than maintenance of the vocabulary/models themselves

## Inputs

- Existing SMetaS vocabulary file for the header in question
- User-confirmed freetext terms absent from the vocabulary (e.g., 'allergen exposure', 'ozone', 'hours/day')

## Outputs

- Expanded SMetaS vocabulary file with new selectable target entries
- Retrained tf-idf term-document matrix over the expanded vocabulary
- Retrained nearest-neighbors index over the tf-idf vectors
- Smoke-test mapping results showing freetext inputs resolve to the new terms

## How to apply

Load the existing SMetaS vocabulary file and append the confirmed freetext terms as selectable target entries. Re-tokenize the expanded vocabulary into n-grams and rebuild the tf-idf term-document matrix over all vocabulary entries, since SMetaS standardizes freetext by synonym matching and typographical fixing in an n-grams/nearest-neighbors model approach. Retrain the nearest-neighbors index on the refreshed tf-idf vectors so the new terms become reachable as mapping targets. Run a smoke-test mapping with sample freetext inputs (e.g., 'allergen exposure', 'ozone', 'hours/day') to confirm they resolve to the newly added selections rather than unrelated vocabulary entries. Persist the expanded vocabulary and retrained model files for use by the containerized SMetaS frontend.

## Related tools

- **SMetaS** (Provides the vocabulary files, tf-idf/nearest-neighbors synonym-matching models, and containerized frontend/API whose models are extended and retrained) — github.com/metabolomics-us/metadatastandardizer
- **MetaS** (Hosts the extensive documentation consulted for running and maintaining the metadata standardizer instance) — https://metabolomics-us.github.io/metadatastandardizer/

## Evaluation signals

- The retrained nearest-neighbors model files exist and are persisted alongside the expanded vocabulary for the containerized SMetaS frontend
- Smoke-test freetext inputs ('allergen exposure', 'ozone', 'hours/day') map to the newly added vocabulary selections rather than unrelated entries
- Pre-existing vocabulary entries still resolve to their original targets after retraining (no regression in synonym/typo matching)
- The expanded vocabulary exposes the new terms as selectable entries for future users

## Limitations

- No quantitative accuracy threshold is reported in the paper; acceptance of the retrained model is a qualitative mapping check only
- The hosted example instance is slow because of low i/o speed on the chosen AWS instance, which can impede interactive testing of the retrained models
- No changelog is available for the repository, so versioning of vocabulary/model changes must be tracked externally

## Evidence

- [abstract] n-grams/nearest-neighbors model basis: "This transformation process is designed to maximize simplicity and is guided by, among other strategies, synonym matching and typographical fixing in an n-grams/nearest neighbors model approach."
- [abstract] containerized deployment target: "SMetaS (Sample Metadata Standardizer) combines a classic database with an API and frontend and is provided in a containerized environment."
- [readme] documentation location: "Extensive documentation is available at https://metabolomics-us.github.io/metadatastandardizer/"
- [readme] hosted instance performance caveat: "This instance is slow because we chose to pay for low i/o speed on our AWS instance. The project can run much faster."
