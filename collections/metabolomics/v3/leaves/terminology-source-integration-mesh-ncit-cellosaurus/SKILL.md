---
name: terminology-source-integration-mesh-ncit-cellosaurus
description: Use when user-submitted freetext metadata strings (sample headers in
  metabolomics submissions) must be mapped onto fixed, curated vocabularies compiled
  from MeSH, NCBI Taxonomy, Cellosaurus, NCIT and FDA sources.
when_to_use_negative:
- When mappings require semantic (non-lexical) reasoning beyond n-gram similarity
  — the approach relies on character-trigram cosine similarity, not ontology semantics
- When the target vocabularies are not among the SMetaS-supported sources (MeSH, NCBI
  Taxonomy, Cellosaurus, NCIT, FDA)
tools:
- name: SMetaS
  role: Provides the vocabulary-generation pipeline and the n-grams/nearest-neighbors
    model used to standardize freetext sample metadata into the merged vocabularies
  repo: https://github.com/metabolomics-us/metadatastandardizer
- name: MetaS
  role: Documented deployment/frontend of the standardizer; extensive documentation
    is hosted online
  repo: https://metabolomics-us.github.io/metadatastandardizer/
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.3390/metabo13080941
    title: 'SMetaS: A Sample Metadata Standardizer for Metabolomics'
  source_content_hash: sha256:9cb33f78d813953e7f1eb20a790f6a7c1d4a38421d0a7f3bf206bb79decb891f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/metabolomics-us/metadatastandardizer
  tools:
  - MetaS
  - SMetaS
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7692307692307693
n_claims: 13
---

# terminology-source-integration-mesh-ncit-cellosaurus

## Summary

Build constrained, non-overlapping per-header vocabularies for SMetaS by merging MeSH, NCBI Taxonomy, Cellosaurus, NCIT, and FDA source dumps, then fit a character-trigram tf-idf vector space and nearest-neighbors model that coerces user freetext into the vocabulary for cosine-similarity mapping.

## When to use

Use when user-submitted freetext metadata strings (sample headers in metabolomics submissions) must be mapped onto fixed, curated vocabularies compiled from MeSH, NCBI Taxonomy, Cellosaurus, NCIT and FDA sources.

## When NOT to use

- When mappings require semantic (non-lexical) reasoning beyond n-gram similarity — the approach relies on character-trigram cosine similarity, not ontology semantics
- When the target vocabularies are not among the SMetaS-supported sources (MeSH, NCBI Taxonomy, Cellosaurus, NCIT, FDA)

## Inputs

- Source vocabulary dumps for MeSH, NCBI Taxonomy, Cellosaurus, NCIT and FDA
- SMetaS vocabulary-generation pipeline (constrained vocabulary construction per metadata header)
- User-submitted freetext metadata strings to be standardized

## Outputs

- One vocabulary file per metadata header (constrained, non-overlapping terms)
- Serialized fitted tf-idf character-trigram vectorizer
- Serialized fitted nearest-neighbors model for freetext-to-term mapping

## How to apply

Download the source vocabulary dumps for MeSH, NCBI Taxonomy, Cellosaurus, NCIT and FDA as used by SMetaS. Run the vocabulary-generation pipeline to build a constrained, non-overlapping vocabulary for each metadata category (header) from the combined sources, writing one vocabulary file per header. Fit the tf-idf character-trigram vectorizer over the vocabulary terms, then fit the nearest-neighbors model on the resulting tf-idf vectors so freetext can be coerced into the pre-determined vector space and mapped to main terms. Serialize the fitted vectorizer and nearest-neighbors model to disk alongside the vocabulary files. Mapping decisions are driven by cosine similarity between trigram tf-idf vectors, supporting synonym matching and typographical fixing of user terms.

## Related tools

- **SMetaS** (Provides the vocabulary-generation pipeline and the n-grams/nearest-neighbors model used to standardize freetext sample metadata into the merged vocabularies) — https://github.com/metabolomics-us/metadatastandardizer
- **MetaS** (Documented deployment/frontend of the standardizer; extensive documentation is hosted online) — https://metabolomics-us.github.io/metadatastandardizer/

## Evaluation signals

- The run exits cleanly after pipeline execution
- A serialized model file (vectorizer + nearest-neighbors) exists on disk
- A vocabulary file exists for every metadata header
- User freetext is coerced into the pre-determined trigram tf-idf vector space and mapped to vocabulary terms via cosine similarity without error
- Vocabularies are constrained and non-overlapping per metadata category

## Limitations

- No quantitative accuracy threshold is reported in the paper; acceptance is defined by successful artifact production (serialized model and per-header vocabulary files)
- The demo AWS instance is slow due to low i/o speed, though the project can run much faster
- No changelog is available for the repository

## Evidence

- [abstract] n-grams/nearest-neighbors model approach: "synonym matching and typographical fixing in an n-grams/nearest neighbors model approach"
- [abstract] FAIRness motivation: "Using such a tool in frontends for metabolomic databases will dramatically increase the FAIRness (Findability, Accessibility, Interoperability, and Reusability) of data"
- [readme] documentation and deployment: "Extensive documentation is available at https://metabolomics-us.github.io/metadatastandardizer/"
