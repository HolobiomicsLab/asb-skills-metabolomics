---
name: rest-api-endpoint-exposure
description: Use when standardized sample metadata produced by the SMetaS freetext-transformation
  pipeline must be consumed programmatically by downstream pipelines or database frontends,
  in parallel with a standardized CSV export.
when_to_use_negative:
- The metadata are already standardized against the fixed vocabularies and only need
  redistribution, not re-exposure via the API.
- Only a file-based handoff is needed and no pipeline integration or programmatic
  query of submitted studies/authors is required.
- The task is the metadata transformation itself (synonym matching, typographical
  fixing) rather than delivery of its results.
edam_operation: http://edamontology.org/operation_2422
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: SMetaS
  role: Provides the containerized combination of database, API, and frontend through
    which the standardized metadata are exposed programmatically.
  repo: github.com/metabolomics-us/metadatastandardizer
- name: MetaS
  role: The deployed implementation whose documentation describes the API/frontend
    surface for standardized metadata.
  repo: https://metabolomics-us.github.io/metadatastandardizer/
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.3390/metabo13080941
    title: 'SMetaS: A Sample Metadata Standardizer for Metabolomics'
  source_content_hash: sha256:92ab2013a7270748b3dde0078039d50421b6ca52a05e6dd6364e4199af016cfb
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MetaS
  - SMetaS
  edam_operation: http://edamontology.org/operation_2422
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# rest-api-endpoint-exposure

## Summary

Exposes SMetaS's standardized sample metadata through a programmatic access point (API) that accepts submitted studies and author information, alongside a standardized CSV export, so both interfaces serve the same curated matrix for downstream pipelines.

## When to use

Use when standardized sample metadata produced by the SMetaS freetext-transformation pipeline must be consumed programmatically by downstream pipelines or database frontends, in parallel with a standardized CSV export.

## When NOT to use

- The metadata are already standardized against the fixed vocabularies and only need redistribution, not re-exposure via the API.
- Only a file-based handoff is needed and no pipeline integration or programmatic query of submitted studies/authors is required.
- The task is the metadata transformation itself (synonym matching, typographical fixing) rather than delivery of its results.

## Inputs

- Curated/standardized sample-metadata matrix (freetext terms replaced with fixed-vocabulary terms via the n-grams/nearest neighbors model)
- Submitted study identifiers and author information

## Outputs

- standardized_metadata.csv (CSV export of the standardized metadata matrix)
- API responses returning standardized metadata for queried studies and authors

## How to apply

Start from the curated sample-metadata matrix produced by SMetaS, in which original freetext entries have been replaced by fixed-vocabulary terms via synonym matching and typographical fixing in an n-grams/nearest neighbors model approach. Serialize that matrix to a standardized CSV (e.g. for study submission or repository deposit), then implement the programmatic access point so it accepts submitted studies and author information and returns the standardized metadata for each submission. Wire both the CSV export and the API response to the same curated matrix so the two interfaces cannot diverge. SMetaS ships as a containerized environment combining 'a classic database with an API and frontend', so deploy the endpoint within that container and point consumers to the project documentation for API details.

## Related tools

- **SMetaS** (Provides the containerized combination of database, API, and frontend through which the standardized metadata are exposed programmatically.) — github.com/metabolomics-us/metadatastandardizer
- **MetaS** (The deployed implementation whose documentation describes the API/frontend surface for standardized metadata.) — https://metabolomics-us.github.io/metadatastandardizer/

## Evaluation signals

- standardized_metadata.csv exists and contains rows with fixed-vocabulary terms in place of the original freetext entries.
- An API query for a submitted study returns metadata rows identical to the corresponding rows in the CSV export.
- An API query for an author returns the standardized metadata associated with that author's submission.
- CSV and API responses resolve to the same underlying curated matrix (no divergence between the two interfaces).
- The endpoint is reachable from within the provided containerized SMetaS environment.

## Limitations

- The hosted example instance is slow because of low AWS i/o speed; the project can run much faster self-hosted, so response-time benchmarks against the demo endpoint are not representative.
- No changelog is available in the article's discussion, so API behavior changes over versions are hard to track.
- The article does not specify authentication, rate limits, or API schema details; consult the external documentation before relying on the endpoint.

## Evidence

- [abstract] SMetaS combines database, API, and frontend in a container: "SMetaS (Sample Metadata Standardizer) combines a classic database with an API and frontend and is provided in a containerized environment."
- [abstract] Transformation guided by synonym matching and typographical fixing: "This transformation process is designed to maximize simplicity and is guided by, among other strategies, synonym matching and typographical fixing in an n-grams/nearest neighbors model approach."
- [readme] Documentation URL for the deployed API/frontend: "Extensive documentation is available at https://metabolomics-us.github.io/metadatastandardizer/"
- [readme] Example hosted instance is slow due to low i/o speed: "This instance is slow because we chose to pay for low i/o speed on our AWS instance. The project can run much faster."
