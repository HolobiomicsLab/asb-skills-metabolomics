---
name: tfidf-nearest-neighbor-term-curation
description: Use when a freetext metabolomics sample-metadata column (csv) must be
  mapped onto fixed SMetaS vocabularies and many strings are near-matches or synonyms
  of existing controlled terms.
when_to_use_negative:
- Numeric metadata strings that are not part of the freetext curation scope (non-numeric
  strings were the target in the ozone use case)
- Metadata already fully encoded as controlled vocabulary terms, where no freetext-to-vocabulary
  transformation is needed
edam_operation: null
edam_topics:
- http://edamontology.org/topic_0219
tools:
- name: SMetaS
  role: Containerized application whose curation pass performs the tf-idf/nearest-neighbor
    freetext-to-vocabulary replacement with synonym matching and typographical fixing
  repo: github.com/metabolomics-us/metadatastandardizer
- name: MetaS
  role: Documentation name for the same metadata standardizer service; extensive docs
    at the GitHub Pages site
  repo: github.com/metabolomics-us/metadatastandardizer
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.3390/metabo13080941
    title: 'SMetaS: A Sample Metadata Standardizer for Metabolomics'
  source_content_hash: sha256:a797a2df9fdee10f4b8a7562f9030ad661ff56defb9078fcebd41f7f161b3a93
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MetaS
  - SMetaS
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0219
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# tfidf-nearest-neighbor-term-curation

## Summary

Curation pass of the SMetaS sample-metadata standardizer that replaces freetext metadata strings with ontology-derived vocabulary terms using an n-grams/nearest neighbors (tf-idf/nnm) model with synonym matching and typographical fixing.

## When to use

Use when a freetext metabolomics sample-metadata column (csv) must be mapped onto fixed SMetaS vocabularies and many strings are near-matches or synonyms of existing controlled terms.

## When NOT to use

- Numeric metadata strings that are not part of the freetext curation scope (non-numeric strings were the target in the ozone use case)
- Metadata already fully encoded as controlled vocabulary terms, where no freetext-to-vocabulary transformation is needed

## Inputs

- Freetext sample metadata csv (e.g., the Figure 3b use-case metadata table for the ozone/house dust mite lung study)
- Initial ontology-derived fixed vocabularies (e.g., zeroTimeEvent, drugName, drugDoseUnit)

## Outputs

- Standardized metadata csv in which freetext terms are replaced by vocabulary terms (Figure 3c-style standardized_metadata.csv)
- List of unmapped non-numeric strings requiring user confirmation as new vocabulary terms

## How to apply

Load the use-case freetext metadata csv (e.g., the Figure 3b ozone/house dust mite lung metabolomics table) into the containerized SMetaS application. Run the n-grams/nearest-neighbors (tf-idf/nnm) curation pass, which converts freetext strings into n-gram vectors, finds nearest neighbors among ontology-derived vocabulary terms, and applies synonym matching and typographical fixing to replace freetext terms with vocabulary terms. Follow with a substring-search pass over the fixed vocabularies to resolve remaining unmapped non-numeric strings. Only strings left unmapped after both passes go to the user-confirmation step for addition as new terms — in the ozone use case this yielded exactly three additions: 'allergen exposure' to zeroTimeEvent, 'ozone' to drugName, and 'hours/day' to drugDoseUnit. Rationale: maximizing simplicity of the transformation while increasing FAIRness of deposited metabolomics data.

## Related tools

- **SMetaS** (Containerized application whose curation pass performs the tf-idf/nearest-neighbor freetext-to-vocabulary replacement with synonym matching and typographical fixing) — github.com/metabolomics-us/metadatastandardizer
- **MetaS** (Documentation name for the same metadata standardizer service; extensive docs at the GitHub Pages site) — github.com/metabolomics-us/metadatastandardizer

## Evaluation signals

- All non-numeric strings in the standardized csv are mapped — no freetext remnants remain
- Exactly the user-confirmed new terms appear under their vocabularies (zeroTimeEvent, drugName, drugDoseUnit in the ozone use case)
- Replaced terms are vocabulary terms reachable via synonym match or typo correction of the original freetext strings
- Remaining unmapped strings after the tf-idf/nnm pass are resolvable by the substring-search pass or flagged for user confirmation

## Limitations

- The tf-idf/nnm pass may leave strings unmapped; a second substring-search pass and a user-confirmation step are needed for residual terms
- No changelog found for the tool per the article's discussion
- The example hosted AWS instance is slow because of low i/o speed, though the project can run much faster locally

## Evidence

- [abstract] SMetaS replaces freetext sample metadata terms with terms from fixed vocabularies using synonym matching and typographical fixing in an n-grams/nearest neighbors model approach.: "This transformation process is designed to maximize simplicity and is guided by, among other strategies, synonym matching and typographical fixing in an n-grams/nearest neighbors model approach."
- [abstract] SMetaS combines a classic database with an API and frontend and is provided in a containerized environment.: "SMetaS (Sample Metadata Standardizer) combines a classic database with an API and frontend and is provided in a containerized environment."
- [readme] Extensive documentation is available at the metabolomics-us GitHub Pages site.: "Extensive documentation is available at https://metabolomics-us.github.io/metadatastandardizer/"
- [readme] The example hosted instance is slow due to low i/o speed on the AWS instance.: "This instance is slow because we chose to pay for low i/o speed on our AWS instance. The project can run much faster."
