---
name: sample-metadata-standardization
description: Use when a metabolomics study's sample metadata is freetext (e.g. zeroTimeEvent,
  drugName, drugDoseUnit columns) and terms must be mapped to fixed vocabularies,
  with the output feeding a study submission, repository deposit, or downstream pipeline.
when_to_use_negative:
- Metadata fields are numeric — SMetaS curation passes target non-numeric freetext
  strings only
- Input metadata already conforms to the target fixed vocabularies (e.g. controlled-terminology
  submissions), leaving nothing to standardize
- The task is general ontology building rather than mapping study metadata to a fixed
  vocabulary
edam_topics: []
tools:
- name: SMetaS
  role: Containerized application (database + API + frontend) that runs the synonym-matching/typographical-fixing
    n-grams/nearest-neighbors curation passes, manages user-confirmed vocabulary additions,
    and serves the CSV export and API.
  repo: github.com/metabolomics-us/metadatastandardizer
- name: MetaS
  role: Documented implementation of the metadata standardizer referenced for usage
    documentation.
  repo: github.com/metabolomics-us/metadatastandardizer
provenance:
  source_task_ids:
  - task_004
  - task_002
  - task_006
  source_papers:
  - doi: 10.3390/metabo13080941
    title: 'SMetaS: A Sample Metadata Standardizer for Metabolomics'
  source_content_hash: sha256:73ce2246d1fc76480f1647688640a3fa725f62d0b80e78d289fd0860b5f6c880
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
yes_rate: 0.6153846153846154
n_claims: 13
---

# sample-metadata-standardization

## Summary

Standardize freetext metabolomics sample metadata by replacing freetext strings with terms from fixed vocabularies via SMetaS's n-grams/nearest-neighbors model (synonym matching and typographical fixing), then export the curated matrix as a standardized CSV and expose it through an API for pipeline integration.

## When to use

Use when a metabolomics study's sample metadata is freetext (e.g. zeroTimeEvent, drugName, drugDoseUnit columns) and terms must be mapped to fixed vocabularies, with the output feeding a study submission, repository deposit, or downstream pipeline.

## When NOT to use

- Metadata fields are numeric — SMetaS curation passes target non-numeric freetext strings only
- Input metadata already conforms to the target fixed vocabularies (e.g. controlled-terminology submissions), leaving nothing to standardize
- The task is general ontology building rather than mapping study metadata to a fixed vocabulary

## Inputs

- freetext sample-metadata CSV (e.g. the ozone/house dust mite lung metabolomics use-case matrix, Figure 3b)
- existing SMetaS fixed vocabulary files (e.g. zeroTimeEvent, drugName, drugDoseUnit)

## Outputs

- standardized_metadata.csv with all non-numeric strings mapped to fixed-vocabulary terms (Figure 3c format)
- expanded vocabulary files containing user-confirmed new terms
- retrained tf-idf/nearest-neighbors model files
- API responses returning standardized metadata for submitted studies and authors

## How to apply

Load the freetext sample-metadata CSV into the containerized SMetaS application, which combines a classic database with an API and frontend. Run the n-grams/nearest-neighbors curation pass (tf-idf/nnm) that applies synonym matching and typographical fixing to replace freetext terms with fixed-vocabulary terms, then run a substring-search pass over the fixed vocabularies for remaining unmapped non-numeric strings. For strings still unmapped after both passes, use the user-confirmation step to add new terms to the corresponding vocabularies (in the ozone/house dust mite lung use case, exactly three were added: 'allergen exposure' to zeroTimeEvent, 'ozone' to drugName, 'hours/day' to drugDoseUnit); user-confirmed additions trigger re-tokenization into n-grams, rebuilding of the tf-idf term-document matrix, and retraining of the nearest-neighbors model so future submissions map to the expanded vocabulary. Serialize the standardized matrix to CSV, and wire both the CSV export and the API (which accepts submitted studies and author information) to the same curated matrix so the two interfaces serve identical standardized content.

## Related tools

- **SMetaS** (Containerized application (database + API + frontend) that runs the synonym-matching/typographical-fixing n-grams/nearest-neighbors curation passes, manages user-confirmed vocabulary additions, and serves the CSV export and API.) — github.com/metabolomics-us/metadatastandardizer
- **MetaS** (Documented implementation of the metadata standardizer referenced for usage documentation.) — github.com/metabolomics-us/metadatastandardizer

## Evaluation signals

- All non-numeric strings in standardized_metadata.csv are mapped — no freetext remnants remain after the n-grams/nearest-neighbors and substring-search passes
- Exactly the user-confirmed new terms appear under their target vocabularies (e.g. 'allergen exposure' in zeroTimeEvent, 'ozone' in drugName, 'hours/day' in drugDoseUnit)
- Smoke-test freetext inputs map to newly added vocabulary terms rather than unrelated entries after model retraining
- A query to the API for a submitted study/author returns the same standardized metadata rows contained in the CSV
- standardized_metadata.csv exists and preserves the fixed-vocabulary terms that replaced original freetext entries

## Limitations

- The paper reports no quantitative accuracy threshold for the mapping — acceptance is a qualitative check that freetext inputs resolve to the intended vocabulary terms
- The public example instance is slow because of low i/o speed on the low-cost AWS instance; running the containerized project locally can be much faster
- Unmapped freetext terms require manual user confirmation before being added and before the tf-idf/nearest-neighbors model is retrained

## Evidence

- [abstract] synonym matching and typographical fixing: "This transformation process is designed to maximize simplicity and is guided by, among other strategies, synonym matching and typographical fixing in an n-grams/nearest neighbors model approach."
- [abstract] SMetaS architecture: "SMetaS (Sample Metadata Standardizer) combines a classic database with an API and frontend and is provided in a containerized environment."
- [readme] documentation location: "Extensive documentation is available at https://metabolomics-us.github.io/metadatastandardizer/"
