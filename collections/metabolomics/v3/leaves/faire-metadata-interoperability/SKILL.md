---
name: faire-metadata-interoperability
description: Use when a standardized sample-metadata matrix (freetext terms replaced
  by fixed-vocabulary terms) must be delivered to users and integrated into existing
  pipelines via CSV export and an API.
when_to_use_negative:
- The metadata matrix has not yet been standardized by the SMetaS n-grams/nearest-neighbors
  vocabulary model — the CSV and API only serve curated, fixed-vocabulary content.
- The target system needs raw freetext metadata preserved rather than terms replaced
  from fixed vocabularies.
edam_topics:
- http://edamontology.org/topic_3334
tools:
- name: SMetaS
  role: The Sample Metadata Standardizer system whose database + API + frontend containerized
    environment delivers the standardized CSV and programmatic access point
- name: MetaS
  role: The documented implementation (metabolomics-us/metadatastandardizer) with
    extensive documentation hosted online for setting up CSV export and API access
  repo: https://metabolomics-us.github.io/metadatastandardizer/
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.3390/metabo13080941
    title: 'SMetaS: A Sample Metadata Standardizer for Metabolomics'
  source_content_hash: sha256:542efb1ac38098f0ae0e0f63d754940a59648bbbe7b80e219e2fd98f76a7b591
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MetaS
  - SMetaS
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3334
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# faire-metadata-interoperability

## Summary

Exposes SMetaS-standardized sample metadata for metabolomics studies through two synchronized interfaces: a standardized CSV file for study submission/repository deposit, and a programmatic API access point so downstream pipelines can retrieve the same curated matrix for submitted studies and authors.

## When to use

Use when a standardized sample-metadata matrix (freetext terms replaced by fixed-vocabulary terms) must be delivered to users and integrated into existing pipelines via CSV export and an API.

## When NOT to use

- The metadata matrix has not yet been standardized by the SMetaS n-grams/nearest-neighbors vocabulary model — the CSV and API only serve curated, fixed-vocabulary content.
- The target system needs raw freetext metadata preserved rather than terms replaced from fixed vocabularies.

## Inputs

- Curated sample-metadata matrix produced by the SMetaS standardization pipeline (synonym-matched, typographically fixed fixed-vocabulary terms)
- Submitted studies and author information accepted by the API

## Outputs

- Standardized CSV file (standardized_metadata.csv) for study submission or repository deposit
- API response returning standardized metadata for each submitted study/author

## How to apply

Load the curated sample-metadata matrix produced by the SMetaS standardization pipeline, in which original freetext entries were replaced by fixed-vocabulary terms via synonym matching and typographical fixing in an n-grams/nearest-neighbors model. Serialize that matrix to a standardized CSV file so users can submit it with a study or deposit it in a repository. Implement the programmatic access point (API) so it accepts submitted studies and author information and returns the standardized metadata for each submission. Wire both the CSV export and the API response to the same curated matrix, so the two interfaces serve identical standardized content — this dual delivery is what SMetaS means by combining a database with an API and frontend in a containerized environment. Validate that the CSV rows and the API response for a given study/author agree exactly before exposing either interface.

## Related tools

- **SMetaS** (The Sample Metadata Standardizer system whose database + API + frontend containerized environment delivers the standardized CSV and programmatic access point)
- **MetaS** (The documented implementation (metabolomics-us/metadatastandardizer) with extensive documentation hosted online for setting up CSV export and API access) — https://metabolomics-us.github.io/metadatastandardizer/

## Evaluation signals

- standardized_metadata.csv exists and contains the fixed-vocabulary terms that replaced the original freetext entries
- A query to the API for a submitted study/author returns metadata rows identical to those contained in the CSV
- Both interfaces (CSV and API) are wired to the same curated matrix, i.e. no divergence between file and service content
- Output terms belong to the fixed vocabularies targeted by the synonym-matching/typographical-fixing model, not the original freetext
- The deployment runs inside the containerized environment combining database, API, and frontend

## Limitations

- The example hosted instance (AWS Elastic Beanstalk) is slow because low i/o speed was chosen for the paid AWS instance; the project can run much faster on better resources.
- No changelog is available for the project (discussion section notes none was found).

## Evidence

- [abstract] SMetaS (Sample Metadata Standardizer) combines a classic database with an API and frontend and is provided in a containerized environment.: "SMetaS (Sample Metadata Standardizer) combines a classic database with an API and frontend and is provided in a containerized environment."
- [abstract] Using such a tool in frontends for metabolomic databases will dramatically increase the FAIRness of data: "Using such a tool in frontends for metabolomic databases will dramatically increase the FAIRness (Findability, Accessibility, Interoperability, and Reusability) of data"
- [abstract] synonym matching and typographical fixing in an n-grams/nearest neighbors model approach: "This transformation process is designed to maximize simplicity and is guided by, among other strategies, synonym matching and typographical fixing in an n-grams/nearest neighbors model approach."
- [readme] Extensive documentation is available at https://metabolomics-us.github.io/metadatastandardizer/: "Extensive documentation is available at https://metabolomics-us.github.io/metadatastandardizer/"
