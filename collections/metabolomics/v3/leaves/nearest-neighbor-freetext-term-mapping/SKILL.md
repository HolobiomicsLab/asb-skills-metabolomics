---
name: nearest-neighbor-freetext-term-mapping
description: Use when freetext sample metadata strings must be coerced into a fixed,
  pre-determined vocabulary via synonym matching and typographical fixing, e.g. curating
  metabolomics metadata headers.
when_to_use_negative:
- When metadata strings already conform to the constrained vocabulary and need no
  term mapping
- When the target vocabularies differ from MeSH, NCBI Taxonomy, Cellosaurus, NCIT
  or FDA and cannot be built by the SMetaS vocabulary-generation pipeline
edam_operation: http://edamontology.org/operation_0282
tools:
- name: SMetaS
  role: Provides the vocabulary-generation pipeline, character-trigram tf-idf vectorizer,
    and nearest-neighbors model used to map freetext metadata to constrained vocabulary
    terms; distributed as a containerized tool with a database, API and frontend
  repo: github.com/metabolomics-us/metadatastandardizer
- name: MetaS
  role: The documented tool behind SMetaS; its documentation site describes installation
    and use of the metadata standardizer
  repo: https://metabolomics-us.github.io/metadatastandardizer/
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.3390/metabo13080941
    title: 'SMetaS: A Sample Metadata Standardizer for Metabolomics'
  source_content_hash: sha256:0d6f4310cf869929c572514ed9c72de590125399d10e42b262d65ac402e6d5e2
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MetaS
  - SMetaS
  edam_operation: http://edamontology.org/operation_0282
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 14
---

# nearest-neighbor-freetext-term-mapping

## Summary

Maps user-submitted freetext sample metadata strings onto constrained vocabulary terms (MeSH, NCBI Taxonomy, Cellosaurus, NCIT, FDA) using a character-trigram tf-idf vector space and nearest-neighbors matching, as implemented in SMetaS for metabolomics metadata standardization.

## When to use

Use when freetext sample metadata strings must be coerced into a fixed, pre-determined vocabulary via synonym matching and typographical fixing, e.g. curating metabolomics metadata headers.

## When NOT to use

- When metadata strings already conform to the constrained vocabulary and need no term mapping
- When the target vocabularies differ from MeSH, NCBI Taxonomy, Cellosaurus, NCIT or FDA and cannot be built by the SMetaS vocabulary-generation pipeline

## Inputs

- Source vocabulary dumps for MeSH, NCBI Taxonomy, Cellosaurus, NCIT and FDA
- User-submitted freetext sample metadata strings per metadata header

## Outputs

- A fitted, serialized tf-idf character-trigram vectorizer
- A fitted, serialized nearest-neighbors model over vocabulary tf-idf vectors
- One constrained, non-overlapping vocabulary file per metadata header
- Mapped vocabulary terms replacing freetext metadata values

## How to apply

First build a constrained, non-overlapping vocabulary per metadata header by running the SMetaS vocabulary-generation pipeline over combined source dumps (MeSH, NCBI Taxonomy, Cellosaurus, NCIT, FDA). Fit a tf-idf vectorizer over the vocabulary terms using character trigrams so new words can be coerced into the pre-determined vector space; then fit a nearest-neighbors model on those tf-idf vectors so cosine similarity determines the distance between vocabulary terms and user-provided terms. Serialize the fitted vectorizer and nearest-neighbors model to disk and write one vocabulary file per metadata header. At runtime, user freetext is vectorized with the serialized tf-idf model and mapped to its nearest main term, achieving synonym matching and typographical fixing in an n-grams/nearest neighbors approach. No quantitative similarity threshold is reported in the paper, so acceptance is the successful production of the serialized model and per-header vocabulary files.

## Related tools

- **SMetaS** (Provides the vocabulary-generation pipeline, character-trigram tf-idf vectorizer, and nearest-neighbors model used to map freetext metadata to constrained vocabulary terms; distributed as a containerized tool with a database, API and frontend) — github.com/metabolomics-us/metadatastandardizer
- **MetaS** (The documented tool behind SMetaS; its documentation site describes installation and use of the metadata standardizer) — https://metabolomics-us.github.io/metadatastandardizer/

## Examples

```
from sklearn.feature_extraction.text import TfidfVectorizer; v = TfidfVectorizer(analyzer='char', ngram_range=(3,3)); X = v.fit_transform(vocab_terms)
```

## Evaluation signals

- The pipeline run exits cleanly
- A serialized model file (vectorizer + nearest-neighbors) exists after the run
- A vocabulary file exists for every metadata header, with no headers missing
- Vocabularies are constrained and non-overlapping per metadata category
- No quantitative similarity threshold is reported in the paper, so artifact production is the acceptance criterion

## Limitations

- No quantitative acceptance threshold is reported in the paper; validation relies on successful artifact production rather than a measured accuracy cutoff
- The example hosted instance is slow because of low i/o speed on the AWS instance, though the project can run much faster
- No changelog was found for the tool

## Evidence

- [abstract] synonym matching and typographical fixing via n-grams/nearest neighbors: "This transformation process is designed to maximize simplicity and is guided by, among other strategies, synonym matching and typographical fixing in an n-grams/nearest neighbors model approach."
- [full_text] cosine similarity over character-trigram tf-idf vector space: [evidence span withheld — anchor status: not_found]
- [readme] SMetaS tool availability and documentation: "Extensive documentation is available at https://metabolomics-us.github.io/metadatastandardizer/"
