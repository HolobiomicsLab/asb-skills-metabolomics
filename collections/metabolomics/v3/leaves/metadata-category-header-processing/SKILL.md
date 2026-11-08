---
name: metadata-category-header-processing
description: Use when freetext sample metadata strings (e.g., organism, cell line,
  tissue headers) must be mapped to constrained vocabulary terms via synonym matching
  and typographical fixing with n-grams/nearest neighbors.
when_to_use_negative:
- Metadata strings are already controlled-vocabulary terms matching the target vocabulary
  — no coercion into the trigram vector space is needed
- The task requires semantic (meaning-based) entity linking rather than character
  n-gram surface similarity; the trigram/cosine approach fixes typos and synonyms
  by string shape, not ontology semantics
- The target domain vocabulary is not among the SMetaS-supported sources (MeSH, NCBI
  Taxonomy, Cellosaurus, NCIT, FDA)
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3077
- http://edamontology.org/topic_0089
tools:
- name: SMetaS
  role: Provides the vocabulary-generation pipeline and the n-grams/nearest-neighbors
    standardization model that this skill reproduces and serializes
  repo: github.com/metabolomics-us/metadatastandardizer
- name: MetaS
  role: Deployed name of the tool whose documentation and containerized distribution
    host the vocabulary files and serialized model
  repo: github.com/metabolomics-us/metadatastandardizer
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.3390/metabo13080941
    title: 'SMetaS: A Sample Metadata Standardizer for Metabolomics'
  source_content_hash: sha256:b9ff0bdf0f8c7cc364cefd5f4e2b6ad7923b15c3d1d30ce6f21bdbea302a2c3c
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MetaS
  - SMetaS
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3077
  - http://edamontology.org/topic_0089
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# metadata-category-header-processing

## Summary

Builds SMetaS constrained vocabularies per metadata header from combined ontology dumps (MeSH, NCBI Taxonomy, Cellosaurus, NCIT, FDA) and fits a character-trigram tf-idf vector space plus nearest-neighbors model so freetext sample metadata can be coerced into fixed vocabulary terms.

## When to use

Use when freetext sample metadata strings (e.g., organism, cell line, tissue headers) must be mapped to constrained vocabulary terms via synonym matching and typographical fixing with n-grams/nearest neighbors.

## When NOT to use

- Metadata strings are already controlled-vocabulary terms matching the target vocabulary — no coercion into the trigram vector space is needed
- The task requires semantic (meaning-based) entity linking rather than character n-gram surface similarity; the trigram/cosine approach fixes typos and synonyms by string shape, not ontology semantics
- The target domain vocabulary is not among the SMetaS-supported sources (MeSH, NCBI Taxonomy, Cellosaurus, NCIT, FDA)

## Inputs

- Source vocabulary dumps for MeSH, NCBI Taxonomy, Cellosaurus, NCIT, and FDA
- Metadata category (header) definitions for metabolomics sample metadata
- User-submitted freetext metadata strings to be mapped to vocabulary terms

## Outputs

- One vocabulary file per metadata header (constrained, non-overlapping terms)
- Serialized fitted tf-idf character-trigram vectorizer
- Serialized fitted nearest-neighbors model mapping freetext to main terms

## How to apply

Download the source vocabulary dumps for MeSH, NCBI Taxonomy, Cellosaurus, NCIT and FDA used by SMetaS, then run the vocabulary-generation pipeline to build a constrained, non-overlapping vocabulary for each metadata category (header) from the combined sources. Fit a character-trigram tf-idf vectorizer over the vocabulary terms so that new user-submitted words are coerced into the pre-determined vector space. Fit a nearest-neighbors model on those tf-idf vectors; cosine similarity determines the distance between vocabulary terms and user-provided terms, enabling synonym matching and typographical fixing. Serialize the fitted vectorizer and nearest-neighbors model to disk and write one vocabulary file per metadata header. Acceptance is the successful production of these artifacts — no quantitative threshold is reported in the paper.

## Related tools

- **SMetaS** (Provides the vocabulary-generation pipeline and the n-grams/nearest-neighbors standardization model that this skill reproduces and serializes) — github.com/metabolomics-us/metadatastandardizer
- **MetaS** (Deployed name of the tool whose documentation and containerized distribution host the vocabulary files and serialized model) — github.com/metabolomics-us/metadatastandardizer

## Evaluation signals

- The pipeline run exits cleanly
- A serialized model file (fitted vectorizer + nearest-neighbors model) exists on disk
- A vocabulary file exists for every metadata header — no header is missing its vocabulary artifact
- Each vocabulary is constrained and non-overlapping across the combined sources
- Held-out freetext strings map to the expected main terms via the fitted nearest-neighbors model

## Limitations

- No quantitative accuracy threshold is reported in the paper, so acceptance is based on artifact production rather than measured mapping performance
- The hosted example instance is slow because low i/o speed was chosen on the AWS instance, though the project can run much faster
- No changelog was found for the tool
- Mapping relies on character-trigram surface similarity, so it may conflate terms that are lexically similar but semantically distinct

## Evidence

- [abstract] SMetaS replaces freetext sample metadata terms with terms from fixed vocabularies using synonym matching and typographical fixing in an n-grams/nearest neighbors model approach.: [evidence span withheld — anchor status: not_found]
- [results] Cosine similarity over trigram tf-idf vectors: [evidence span withheld — anchor status: not_found]
- [readme] Containerized distribution and docs: "Extensive documentation is available at https://metabolomics-us.github.io/metadatastandardizer/"
- [abstract] FAIRness motivation: "Using such a tool in frontends for metabolomic databases will dramatically increase the FAIRness (Findability, Accessibility, Interoperability, and Reusability) of data"
