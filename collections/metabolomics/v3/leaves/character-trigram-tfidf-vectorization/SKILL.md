---
name: character-trigram-tfidf-vectorization
description: Use when user-submitted freetext sample metadata strings must be mapped
  onto terms of a constrained, non-overlapping vocabulary (e.g., MeSH, NCBI Taxonomy,
  Cellosaurus, NCIT, FDA) via character n-gram similarity.
when_to_use_negative:
- When exact-match or synonym-table lookup is sufficient and no typographical fixing
  in an n-grams/nearest neighbors model approach is needed
- When the metadata category has no pre-determined constrained vocabulary to map terms
  into
edam_operation: http://edamontology.org/operation_0306
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: SMetaS
  role: Provides the vocabulary-generation pipeline and the tf-idf character-trigram
    vectorizer plus nearest-neighbors model used to map freetext metadata to constrained
    vocabulary terms
  repo: github.com/metabolomics-us/metadatastandardizer
- name: MetaS
  role: Documented deployment of the standardizer whose n-grams/nearest neighbors
    model implements this vectorization skill
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.3390/metabo13080941
    title: 'SMetaS: A Sample Metadata Standardizer for Metabolomics'
  source_content_hash: sha256:900c42b8866cf8afe315735ef5991ec751be15593a4e019030d691ae064034a2
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MetaS
  - SMetaS
  edam_operation: http://edamontology.org/operation_0306
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 16
---

# character-trigram-tfidf-vectorization

## Summary

A vocabulary-matching technique used by SMetaS in which vocabulary terms and user-submitted freetext metadata strings are represented in a character-trigram tf-idf vector space, and cosine similarity via a nearest-neighbors model maps freetext to constrained vocabulary terms.

## When to use

Use when user-submitted freetext sample metadata strings must be mapped onto terms of a constrained, non-overlapping vocabulary (e.g., MeSH, NCBI Taxonomy, Cellosaurus, NCIT, FDA) via character n-gram similarity.

## When NOT to use

- When exact-match or synonym-table lookup is sufficient and no typographical fixing in an n-grams/nearest neighbors model approach is needed
- When the metadata category has no pre-determined constrained vocabulary to map terms into

## Inputs

- Vocabulary source dumps for MeSH, NCBI Taxonomy, Cellosaurus, NCIT and FDA
- Generated vocabulary files (one per metadata header)
- User-submitted freetext metadata strings

## Outputs

- Fitted character-trigram tf-idf vectorizer serialized to disk
- Fitted nearest-neighbors model serialized to disk
- Vocabulary file per metadata header
- Mapping of user freetext terms to curated vocabulary main terms

## How to apply

First download the source vocabulary dumps (MeSH, NCBI Taxonomy, Cellosaurus, NCIT, FDA) and run the SMetaS vocabulary-generation pipeline to build a constrained, non-overlapping vocabulary per metadata header. Then fit a tf-idf character-trigram vectorizer over the vocabulary terms so new words are coerced into the pre-determined vector space. Fit a nearest-neighbors model on those tf-idf vectors so user freetext can be mapped to main terms; cosine similarity determines the distance between vocabulary terms and user-provided terms, supporting synonym matching and typographical fixing. Finally serialize the fitted vectorizer and nearest-neighbors model to disk and write one vocabulary file per metadata header so the mapping is reproducible and deployable.

## Related tools

- **SMetaS** (Provides the vocabulary-generation pipeline and the tf-idf character-trigram vectorizer plus nearest-neighbors model used to map freetext metadata to constrained vocabulary terms) — github.com/metabolomics-us/metadatastandardizer
- **MetaS** (Documented deployment of the standardizer whose n-grams/nearest neighbors model implements this vectorization skill)

## Evaluation signals

- The vocabulary-generation pipeline run exits cleanly
- A serialized model file exists for every metadata header
- A vocabulary file exists for every metadata header
- User-submitted freetext strings are successfully coerced into the pre-determined vector space and matched to main terms via cosine similarity
- No quantitative accuracy threshold is reported in the paper, so acceptance is the successful production of these artifacts

## Limitations

- No quantitative threshold is reported in the paper for mapping quality; acceptance is judged by artifact production rather than accuracy metrics
- The example deployment is slow because low i/o speed was chosen on the AWS instance, though the project can run much faster
- No changelog is available for the tool

## Evidence

- [abstract] abstract quote: "This transformation process is designed to maximize simplicity and is guided by, among other strategies, synonym matching and typographical fixing in an n-grams/nearest neighbors model approach."
- [methods] methods quote: [evidence span withheld — anchor status: not_found]
- [readme] readme quote: "Extensive documentation is available at https://metabolomics-us.github.io/metadatastandardizer/"
