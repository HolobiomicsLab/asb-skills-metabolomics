---
name: synonym-matching-vocabulary-mapping
description: Use when a metabolomics sample-metadata matrix contains freetext field
  values (inconsistent synonyms, typographical errors) that must be replaced with
  fixed-vocabulary terms to make datasets interoperable and FAIR.
when_to_use_negative:
- Input metadata is already expressed in fixed-vocabulary terms with no synonym or
  typo variance — the transformation adds no value and risks unnecessary term churn.
- The task is general text normalization outside sample-metadata fields (e.g. free-form
  abstracts or prose) — the n-grams/nearest-neighbors model targets metadata vocabulary
  terms, not arbitrary prose.
- A downstream system requires the original unmodified freetext provenance — replacement
  by fixed-vocabulary terms discards the original strings.
edam_topics:
- http://edamontology.org/topic_3391
- http://edamontology.org/topic_0091
tools:
- name: SMetaS
  role: Performs the synonym-matching and typographical-fixing transformation of freetext
    metadata into fixed-vocabulary terms, then exposes results via CSV and API
  repo: https://github.com/metabolomics-us/metadatastandardizer
- name: MetaS
  role: Implementation/documentation entry point for the metadata standardizer referenced
    in the article intro
  repo: https://metabolomics-us.github.io/metadatastandardizer/
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.3390/metabo13080941
    title: 'SMetaS: A Sample Metadata Standardizer for Metabolomics'
  source_content_hash: sha256:7bd5830b4ff49f202b529dedab02f6129a6ea4f92d1ffff276f51c9af0b18867
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/metabolomics-us/metadatastandardizer
  tools:
  - MetaS
  - SMetaS
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3391
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 12
---

# synonym-matching-vocabulary-mapping

## Summary

SMetaS's core transformation maps freetext sample-metadata terms in a metabolomics metadata matrix onto fixed vocabularies using synonym matching and typographical fixing in an n-grams/nearest neighbors model approach, yielding standardized metadata suitable for study submission or repository deposit.

## When to use

Use when a metabolomics sample-metadata matrix contains freetext field values (inconsistent synonyms, typographical errors) that must be replaced with fixed-vocabulary terms to make datasets interoperable and FAIR.

## When NOT to use

- Input metadata is already expressed in fixed-vocabulary terms with no synonym or typo variance — the transformation adds no value and risks unnecessary term churn.
- The task is general text normalization outside sample-metadata fields (e.g. free-form abstracts or prose) — the n-grams/nearest-neighbors model targets metadata vocabulary terms, not arbitrary prose.
- A downstream system requires the original unmodified freetext provenance — replacement by fixed-vocabulary terms discards the original strings.

## Inputs

- freetext sample-metadata matrix from a metabolomics study (user-submitted study and author metadata)
- fixed vocabularies used for term replacement

## Outputs

- standardized metadata matrix with freetext entries replaced by fixed-vocabulary terms
- standardized_metadata.csv export for study submission or repository deposit
- API response returning the standardized metadata for submitted studies and authors

## How to apply

Load the user's freetext metadata matrix and run the SMetaS transformation, which is 'designed to maximize simplicity' and guided by synonym matching plus typographical fixing within an n-grams/nearest neighbors model — the model matches each freetext token to candidate vocabulary terms and corrects typos so entries converge on controlled vocabulary terms. Run the standardizer in its containerized environment (SMetaS combines a classic database with an API and frontend). Serialize the curated matrix to a standardized CSV preserving the fixed-vocabulary replacements, and expose the same curated matrix through the programmatic API that accepts submitted studies and authors. Wire both the CSV export and the API response to the identical curated matrix so downstream pipelines and manual submissions see the same standardized content.

## Related tools

- **SMetaS** (Performs the synonym-matching and typographical-fixing transformation of freetext metadata into fixed-vocabulary terms, then exposes results via CSV and API) — https://github.com/metabolomics-us/metadatastandardizer
- **MetaS** (Implementation/documentation entry point for the metadata standardizer referenced in the article intro) — https://metabolomics-us.github.io/metadatastandardizer/

## Evaluation signals

- Verify that standardized_metadata.csv exists after the run.
- Query the API for a submitted study/author and confirm it returns the same standardized metadata rows contained in the CSV (both interfaces serve identical curated content).
- Check that no original freetext term survives where a fixed-vocabulary replacement was available — output entries should belong to the fixed vocabularies.
- Confirm known synonyms of the same concept map to a single canonical vocabulary term across the matrix.
- Confirm obvious typographical variants (e.g. misspelled field values) are corrected by the n-grams/nearest-neighbors model rather than left unchanged.

## Limitations

- The example hosted instance at the AWS Elastic Beanstalk URL is slow because of low I/O speed on the chosen AWS instance; the project can run much faster when self-hosted.
- No changelog is available in the repository, so version drift of vocabularies or matching behavior is hard to track.
- The paper notes the transformation is 'designed to maximize simplicity', implying aggressive simplification rather than rich, context-sensitive disambiguation.

## Evidence

- [abstract] transformation guided by synonym matching and typo fixing: "This transformation process is designed to maximize simplicity and is guided by, among other strategies, synonym matching and typographical fixing in an n-grams/nearest neighbors model approach."
- [abstract] SMetaS architecture: database, API, frontend, container: "SMetaS (Sample Metadata Standardizer) combines a classic database with an API and frontend and is provided in a containerized environment."
- [abstract] FAIRness rationale for applying the standardizer: "Using such a tool in frontends for metabolomic databases will dramatically increase the FAIRness (Findability, Accessibility, Interoperability, and Reusability) of data"
- [intro] documentation location for the standardizer: "Extensive documentation is available at https://metabolomics-us.github.io/metadatastandardizer/"
- [readme] hosted example instance and performance caveat: "This instance is slow because we chose to pay for low i/o speed on our AWS instance. The project can run much faster."
