---
name: controlled-vocabulary-construction
description: Use when user-submitted freetext sample metadata must be standardized
  against fixed vocabularies, or when you need to construct non-overlapping term sets
  per metadata header and train a term-mapping model from source ontology dumps.
when_to_use_negative:
- Metadata are already constrained to the target vocabulary and only need validation
  rather than term mapping
- The metadata category requires semantic/ontology-aware disambiguation rather than
  string similarity via character-trigram tf-idf nearest neighbors
tools:
- name: SMetaS
  role: Provides the vocabulary-generation pipeline and the tf-idf character-trigram
    + nearest-neighbors term-mapping machinery that this skill constructs and serializes
  repo: github.com/metabolomics-us/metadatastandardizer
- name: MetaS
  role: Documented frontend/API deployment of the standardizer that consumes the generated
    vocabularies and serialized model
  repo: https://metabolomics-us.github.io/metadatastandardizer/
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.3390/metabo13080941
    title: 'SMetaS: A Sample Metadata Standardizer for Metabolomics'
  source_content_hash: sha256:4b0e6df6e13a3ee7b5b2f1aa789a70aac39d9e3cdcaee964fd37d81839ed33b4
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

# controlled-vocabulary-construction

## Summary

Builds constrained, non-overlapping vocabularies per metadata category from source dumps (MeSH, NCBI Taxonomy, Cellosaurus, NCIT, FDA) and fits a character-trigram tf-idf vector space plus a nearest-neighbors model so freetext metadata strings can be mapped to standard vocabulary terms, as implemented in SMetaS.

## When to use

Use when user-submitted freetext sample metadata must be standardized against fixed vocabularies, or when you need to construct non-overlapping term sets per metadata header and train a term-mapping model from source ontology dumps.

## When NOT to use

- Metadata are already constrained to the target vocabulary and only need validation rather than term mapping
- The metadata category requires semantic/ontology-aware disambiguation rather than string similarity via character-trigram tf-idf nearest neighbors

## Inputs

- Source vocabulary dumps for MeSH, NCBI Taxonomy, Cellosaurus, NCIT, and FDA
- User-submitted freetext metadata strings for each metadata category (header)

## Outputs

- One constrained, non-overlapping vocabulary file per metadata header
- Serialized tf-idf character-trigram vectorizer
- Serialized nearest-neighbors model mapping freetext to main terms

## How to apply

Download the source vocabulary dumps for MeSH, NCBI Taxonomy, Cellosaurus, NCIT and FDA that SMetaS draws from. Run the SMetaS vocabulary-generation pipeline to build a constrained, non-overlapping vocabulary for each metadata category (header) from the combined sources, writing one vocabulary file per metadata header. Fit the tf-idf character-trigram vectorizer over the vocabulary terms so that new words are coerced into the pre-determined vector space, then fit a nearest-neighbors model on those vectors so that cosine similarity determines the distance between vocabulary terms and user-provided terms. Serialize the fitted vectorizer plus nearest-neighbors model to disk; at standardization time, SMetaS uses synonym matching and typographical fixing in this n-grams/nearest neighbors approach to replace freetext with fixed-vocabulary terms.

## Related tools

- **SMetaS** (Provides the vocabulary-generation pipeline and the tf-idf character-trigram + nearest-neighbors term-mapping machinery that this skill constructs and serializes) — github.com/metabolomics-us/metadatastandardizer
- **MetaS** (Documented frontend/API deployment of the standardizer that consumes the generated vocabularies and serialized model) — https://metabolomics-us.github.io/metadatastandardizer/

## Evaluation signals

- The pipeline run exits cleanly
- A serialized model file (fitted vectorizer + nearest-neighbors) exists
- A vocabulary file exists for every metadata header
- No quantitative acceptance threshold is reported in the paper, so successful production of these artifacts is the acceptance criterion

## Limitations

- The paper reports no quantitative accuracy threshold for the term mapping; acceptance is defined by successful artifact production
- The hosted example instance is slow because low I/O speed was chosen on the AWS instance; the project can run much faster
- Vocabularies are pre-determined — new user words are coerced into the existing vector space rather than added to it
- No changelog is available for tracking vocabulary or model revisions

## Evidence

- [abstract] n-grams/nearest neighbors term mapping: "This transformation process is designed to maximize simplicity and is guided by, among other strategies, synonym matching and typographical fixing in an n-grams/nearest neighbors model approach."
- [abstract] SMetaS packaging: "SMetaS (Sample Metadata Standardizer) combines a classic database with an API and frontend and is provided in a containerized environment."
- [readme] SMetaS documentation location: "Extensive documentation is available at https://metabolomics-us.github.io/metadatastandardizer/"
