---
name: typographical-error-tolerance-matching
description: Use when sample metadata contains freetext terms (e.g. 'allergen exposure',
  'ozone', 'hours/day') that must be resolved to standardized vocabulary entries despite
  synonyms or typographical errors, or when newly confirmed terms must be added and
  models retrained.
when_to_use_negative:
- Input metadata already conforms to a fixed vocabulary, so no synonym matching or
  typographical fixing is needed
- The freetext term is not user-confirmed and has no clear correspondence to an existing
  vocabulary entry, risking mappings to unrelated entries
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: SMetaS
  role: Performs freetext-to-vocabulary mapping via n-grams/nearest-neighbors tf-idf
    models for synonym matching and typographical fixing; vocabulary and models are
    retrained when new terms are confirmed
  repo: github.com/metabolomics-us/metadatastandardizer
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.3390/metabo13080941
    title: 'SMetaS: A Sample Metadata Standardizer for Metabolomics'
  source_content_hash: sha256:a1028d2130992e6c89e75bfb8d5d218c3164a8e2d72963ac8485fc0afe4e9c46
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - SMetaS
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# typographical-error-tolerance-matching

## Summary

Maps user-supplied freetext sample metadata terms to fixed vocabulary entries in SMetaS using synonym matching and typographical fixing within an n-grams/nearest-neighbors model over tf-idf term vectors, and retrains the model when users confirm new vocabulary terms.

## When to use

Use when sample metadata contains freetext terms (e.g. 'allergen exposure', 'ozone', 'hours/day') that must be resolved to standardized vocabulary entries despite synonyms or typographical errors, or when newly confirmed terms must be added and models retrained.

## When NOT to use

- Input metadata already conforms to a fixed vocabulary, so no synonym matching or typographical fixing is needed
- The freetext term is not user-confirmed and has no clear correspondence to an existing vocabulary entry, risking mappings to unrelated entries

## Inputs

- Existing SMetaS vocabulary file (fixed vocabularies of metadata terms)
- User-confirmed freetext terms to add (e.g. 'allergen exposure', 'ozone', 'hours/day')
- Freetext sample metadata terms submitted through the SMetaS frontend/API

## Outputs

- Expanded vocabulary file with new selectable target entries
- Retrained tf-idf term-document matrix and nearest-neighbors model files
- Mapping of freetext input terms to standardized vocabulary selections

## How to apply

Load the existing SMetaS vocabulary file and append user-confirmed freetext terms as selectable target entries. Re-tokenize the expanded vocabulary into n-grams and rebuild the tf-idf term-document matrix over all vocabulary entries, then retrain the nearest-neighbors index (used for synonym matching and typographical fixing) on the refreshed matrix. Persist the expanded vocabulary and retrained model files for use by the containerized SMetaS frontend, then run smoke-test mappings with sample freetext inputs to confirm they resolve to the newly added vocabulary selections rather than unrelated entries.

## Related tools

- **SMetaS** (Performs freetext-to-vocabulary mapping via n-grams/nearest-neighbors tf-idf models for synonym matching and typographical fixing; vocabulary and models are retrained when new terms are confirmed) — github.com/metabolomics-us/metadatastandardizer

## Evaluation signals

- Retrained nearest-neighbors model and expanded vocabulary files exist and are persisted for the containerized frontend
- Smoke-test freetext inputs map to the newly added terms ('allergen exposure', 'ozone', 'hours/day') rather than unrelated vocabulary entries
- New terms appear as selectable target entries in the vocabulary used by future users
- tf-idf term-document matrix is rebuilt over the full expanded vocabulary, not only new entries
- No quantitative threshold is reported in the paper, so acceptance is the qualitative mapping check above

## Limitations

- No quantitative accuracy threshold is reported; validation is a qualitative smoke-test mapping check
- Only user-confirmed terms are added to the vocabulary, so unconfirmed freetext terms do not extend coverage
- The hosted example instance is slow by design due to low I/O speed on the AWS instance; performance depends on deployment configuration

## Evidence

- [abstract] n-grams/nearest-neighbors model approach: [evidence span withheld — anchor status: not_found]
- [abstract] containerized environment: "SMetaS (Sample Metadata Standardizer) combines a classic database with an API and frontend and is provided in a containerized environment."
- [readme] documentation and example instance: "Extensive documentation is available at https://metabolomics-us.github.io/metadatastandardizer/ An example website is hosted at http://metasmetas.us-west-2.elasticbeanstalk.com/."
