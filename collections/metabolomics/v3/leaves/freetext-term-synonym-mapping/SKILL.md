---
name: freetext-term-synonym-mapping
description: Use when sample metadata contains freetext descriptions that must be
  replaced by terms from a fixed vocabulary, or when a user has confirmed adding a
  new freetext term absent from an existing vocabulary.
when_to_use_negative:
- The metadata is already composed of controlled-vocabulary terms from the fixed vocabularies,
  so no synonym matching or typographical fixing is needed
- The goal is FAIRness assessment of an existing database rather than transforming
  freetext metadata into fixed terms
- A quantitative accuracy threshold is required for acceptance; the paper validates
  only via qualitative smoke-test mapping with no reported threshold
edam_operation: http://edamontology.org/operation_0306
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: SMetaS
  role: Combines a classic database with an API and frontend in a containerized environment;
    its n-grams/nearest-neighbors model performs synonym matching and typographical
    fixing of freetext metadata terms, and is retrained after vocabulary expansion
  repo: github.com/metabolomics-us/metadatastandardizer
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.3390/metabo13080941
    title: 'SMetaS: A Sample Metadata Standardizer for Metabolomics'
  source_content_hash: sha256:08226573e03afd2dc63f27b1a98121b4834d9d9a76c9146cc8195bb9ae5c9cb2
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - SMetaS
  edam_operation: http://edamontology.org/operation_0306
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# freetext-term-synonym-mapping

## Summary

Maps freetext sample metadata terms (e.g. 'allergen exposure', 'ozone', 'hours/day') to fixed controlled-vocabulary entries using SMetaS's n-grams/tf-idf/nearest-neighbors approach for synonym matching and typographical fixing; confirmed new terms are appended to the vocabulary and models retrained.

## When to use

Use when sample metadata contains freetext descriptions that must be replaced by terms from a fixed vocabulary, or when a user has confirmed adding a new freetext term absent from an existing vocabulary.

## When NOT to use

- The metadata is already composed of controlled-vocabulary terms from the fixed vocabularies, so no synonym matching or typographical fixing is needed
- The goal is FAIRness assessment of an existing database rather than transforming freetext metadata into fixed terms
- A quantitative accuracy threshold is required for acceptance; the paper validates only via qualitative smoke-test mapping with no reported threshold

## Inputs

- SMetaS vocabulary file of fixed controlled terms (e.g. sample metadata vocabularies)
- User-confirmed new freetext terms not present in the vocabulary (e.g. 'allergen exposure', 'ozone', 'hours/day')
- Freetext sample metadata fields from users or database frontends

## Outputs

- Expanded vocabulary file with new terms as selectable target entries
- Retrained tf-idf/nearest-neighbors model files over the expanded vocabulary
- Standardized metadata terms replacing the freetext inputs, persisted for the containerized SMetaS frontend

## How to apply

Load the existing SMetaS vocabulary file and append user-confirmed freetext terms as selectable target entries. Re-tokenize the expanded vocabulary into n-grams and rebuild the tf-idf term-document matrix over all vocabulary entries. Retrain the nearest-neighbors index over the refreshed tf-idf vectors, which SMetaS uses for synonym matching and typographical fixing. Run a smoke-test mapping with sample freetext inputs to confirm they resolve to the newly added vocabulary selections (e.g. 'allergen exposure', 'ozone', 'hours/day'), then persist the expanded vocabulary and retrained model files for the containerized SMetaS frontend. Acceptance is qualitative: smoke-test inputs must map to the newly added terms rather than unrelated vocabulary entries, since the paper reports no quantitative threshold.

## Related tools

- **SMetaS** (Combines a classic database with an API and frontend in a containerized environment; its n-grams/nearest-neighbors model performs synonym matching and typographical fixing of freetext metadata terms, and is retrained after vocabulary expansion) — github.com/metabolomics-us/metadatastandardizer

## Evaluation signals

- Smoke-test freetext inputs such as 'allergen exposure', 'ozone', and 'hours/day' resolve to the newly added vocabulary entries rather than unrelated terms
- The retrained nearest-neighbors model over the refreshed tf-idf matrix exists and is the file persisted for the containerized frontend
- The expanded vocabulary file contains the confirmed new terms as selectable target entries alongside the original fixed vocabularies
- Typographically incorrect freetext input variants still resolve to the intended vocabulary term via the typographical-fixing path

## Limitations

- No quantitative threshold is reported in the paper; acceptance of retrained models relies on qualitative smoke-test mapping checks
- The public AWS example instance is slow because of low i/o speed on the hosted AWS instance
- No changelog is available for the project, so tracking vocabulary/model version history is not supported out of the box

## Evidence

- [abstract] synonym matching and typographical fixing in an n-grams/nearest neighbors model approach: "This transformation process is designed to maximize simplicity and is guided by, among other strategies, synonym matching and typographical fixing in an n-grams/nearest neighbors model approach."
- [abstract] SMetaS combines a classic database with an API and frontend and is provided in a containerized environment: "SMetaS (Sample Metadata Standardizer) combines a classic database with an API and frontend and is provided in a containerized environment."
- [readme] Extensive documentation is available at the project docs URL: "Extensive documentation is available at https://metabolomics-us.github.io/metadatastandardizer/"
