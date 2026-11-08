---
name: metadata-vocabulary-substring-matching
description: Use when a sample metadata table (e.g., a CSV of freetext fields like
  zeroTimeEvent, drugName, drugDoseUnit) contains non-numeric freetext strings that
  must be mapped to fixed, ontology-derived vocabularies for FAIR reuse.
when_to_use_negative:
- The metadata field is numeric (e.g., dose values or time points) — the transformation
  targets non-numeric freetext strings only
- The input metadata is already standardized against the target vocabularies, leaving
  no freetext remnants to map
- The goal is general text mining of unstructured documents rather than mapping discrete
  metadata field values to a fixed vocabulary
edam_operation: http://edamontology.org/operation_0306
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: SMetaS
  role: Containerized application (classic database + API + frontend) that executes
    the three-pass freetext-to-vocabulary transformation, including the n-grams/nearest-neighbors
    curation pass, substring-search pass, and user-confirmation step for new terms
  repo: https://github.com/metabolomics-us/metadatastandardizer
- name: MetaS
  role: The underlying documentation/project for SMetaS; documentation at the GitHub
    Pages site guides running the standardization passes and hosting an example instance
  repo: https://github.com/metabolomics-us/metadatastandardizer
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.3390/metabo13080941
    title: 'SMetaS: A Sample Metadata Standardizer for Metabolomics'
  source_content_hash: sha256:37d3d39ac356d7d40f713e92ab4ed481b97913c5c1cec25a33b4128b23f7b85f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/metabolomics-us/metadatastandardizer
  tools:
  - MetaS
  - SMetaS
  edam_operation: http://edamontology.org/operation_0306
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# metadata-vocabulary-substring-matching

## Summary

A three-pass curation procedure, implemented in the containerized SMetaS/MetaS application, that transforms freetext sample metadata strings into ontology-derived vocabulary terms via tf-idf n-grams/nearest-neighbor matching, substring search over fixed vocabularies, and user-confirmed creation of genuinely new terms.

## When to use

Use when a sample metadata table (e.g., a CSV of freetext fields like zeroTimeEvent, drugName, drugDoseUnit) contains non-numeric freetext strings that must be mapped to fixed, ontology-derived vocabularies for FAIR reuse.

## When NOT to use

- The metadata field is numeric (e.g., dose values or time points) — the transformation targets non-numeric freetext strings only
- The input metadata is already standardized against the target vocabularies, leaving no freetext remnants to map
- The goal is general text mining of unstructured documents rather than mapping discrete metadata field values to a fixed vocabulary

## Inputs

- CSV file of freetext sample metadata (e.g., the Figure 3b use-case matrix for the ozone/house dust mite lung metabolomics study)
- Initial ontology-derived fixed vocabularies (e.g., zeroTimeEvent, drugName, drugDoseUnit term lists)

## Outputs

- standardized_metadata.csv — metadata table whose non-numeric strings are all mapped vocabulary terms (Figure 3c output)
- Updated vocabulary files containing user-confirmed new terms added to zeroTimeEvent, drugName, and drugDoseUnit

## How to apply

Load the freetext metadata CSV (e.g., the Figure 3b use-case matrix) into the containerized SMetaS application. Run the first curation pass, which applies synonym matching and typographical fixing through an n-grams/nearest-neighbors (tf-idf/nnm) model to replace freetext with vocabulary terms. Run a second pass as a substring search over the fixed vocabularies to resolve remaining unmapped non-numeric strings. For any strings still unmapped after both passes, invoke the user-confirmation step to add exactly those strings as new terms — in the ozone/house dust mite lung metabolomics use case this added 'allergen exposure' to zeroTimeEvent, 'ozone' to drugName, and 'hours/day' to drugDoseUnit. Export the standardized table (Figure 3c CSV) and verify no freetext remnants remain among non-numeric entries.

## Related tools

- **SMetaS** (Containerized application (classic database + API + frontend) that executes the three-pass freetext-to-vocabulary transformation, including the n-grams/nearest-neighbors curation pass, substring-search pass, and user-confirmation step for new terms) — https://github.com/metabolomics-us/metadatastandardizer
- **MetaS** (The underlying documentation/project for SMetaS; documentation at the GitHub Pages site guides running the standardization passes and hosting an example instance) — https://github.com/metabolomics-us/metadatastandardizer

## Evaluation signals

- All non-numeric strings in the exported standardized CSV are mapped vocabulary terms — no freetext remnants remain
- Exactly three user-confirmed new terms ('allergen exposure', 'ozone', 'hours/day') appear under the zeroTimeEvent, drugName, and drugDoseUnit vocabularies for the ozone/lung use case
- Every string reachable by synonym matching or substring search was resolved by the automated passes, not the confirmation step
- The exported table corresponds to the Figure 3c standardized metadata, reproducible by running the three passes on the Figure 3b input matrix

## Limitations

- No changelog was found in the article's discussion, so version-to-version behavior of the matching passes is undocumented
- The hosted example instance (AWS Elastic Beanstalk) is slow due to low i/o speed; the project can run much faster when self-hosted
- Genuinely novel freetext strings cannot be resolved automatically and require the user-confirmation step to be added as new vocabulary terms

## Evidence

- [abstract] SMetaS replaces freetext sample metadata terms with terms from fixed vocabularies using synonym matching and typographical fixing in an n-grams/nearest neighbors model approach.: [evidence span withheld — anchor status: not_found]
- [abstract] Containerized deployment with database, API, and frontend.: "SMetaS (Sample Metadata Standardizer) combines a classic database with an API and frontend and is provided in a containerized environment."
- [other] Ozone/lung use case: nearly all non-numeric strings already mapped; three new terms added.: [evidence span withheld — anchor status: not_found]
- [readme] README pointer to extensive MetaS/MetaS documentation.: "Extensive documentation is available at https://metabolomics-us.github.io/metadatastandardizer/"
