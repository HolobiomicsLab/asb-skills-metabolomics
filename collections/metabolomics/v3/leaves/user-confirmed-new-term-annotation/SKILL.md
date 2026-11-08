---
name: user-confirmed-new-term-annotation
description: Use when after running SMetaS's tf-idf/nearest-neighbors curation pass
  and the substring-search pass over the fixed vocabularies, non-numeric freetext
  strings (e.g. 'allergen exposure', 'ozone', 'hours/day') remain unmapped and must
  be added to specific vocabularies via user confirmation.
when_to_use_negative:
- When freetext strings are already resolvable by the automated synonym-matching/typographical-fixing
  (n-grams/nearest-neighbors) pass or the substring-search pass — only strings still
  unmapped after both passes should reach this step
- When metadata values are numeric rather than freetext strings — the workflow targets
  non-numeric metadata strings
- When the goal is wholesale vocabulary extension without reviewing individual mappings
  — this skill requires per-term user confirmation, not bulk import
edam_operation: http://edamontology.org/operation_0226
edam_topics:
- http://edamontology.org/topic_3383
tools:
- name: SMetaS
  role: Containerized application (database + API + frontend) in which the two automated
    mapping passes run and whose user-confirmation step is used to add the new terms
    to the fixed vocabularies before exporting the standardized csv
  repo: https://github.com/metabolomics-us/metadatastandardizer
- name: MetaS
  role: The underlying MetaS/MetaS tool whose extensive documentation covers the metadata
    standardization workflow executed here
  repo: https://metabolomics-us.github.io/metadatastandardizer/
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.3390/metabo13080941
    title: 'SMetaS: A Sample Metadata Standardizer for Metabolomics'
  source_content_hash: sha256:6683a7496209b2e8727722322faa55eeadac05bcc7bb7d5f77126871af9b1935
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/metabolomics-us/metadatastandardizer
  tools:
  - MetaS
  - SMetaS
  edam_operation: http://edamontology.org/operation_0226
  edam_topics:
  - http://edamontology.org/topic_3383
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# user-confirmed-new-term-annotation

## Summary

A final curation step in the SMetaS metadata standardization workflow where freetext strings that survive the n-grams/nearest-neighbors and substring-search passes are presented to the user for confirmation and added as new terms to the fixed vocabularies. It guarantees complete mapping of non-numeric metadata while keeping vocabulary growth explicit and human-approved.

## When to use

Use when, after running SMetaS's tf-idf/nearest-neighbors curation pass and the substring-search pass over the fixed vocabularies, non-numeric freetext strings (e.g. 'allergen exposure', 'ozone', 'hours/day') remain unmapped and must be added to specific vocabularies via user confirmation.

## When NOT to use

- When freetext strings are already resolvable by the automated synonym-matching/typographical-fixing (n-grams/nearest-neighbors) pass or the substring-search pass — only strings still unmapped after both passes should reach this step
- When metadata values are numeric rather than freetext strings — the workflow targets non-numeric metadata strings
- When the goal is wholesale vocabulary extension without reviewing individual mappings — this skill requires per-term user confirmation, not bulk import

## Inputs

- Freetext sample metadata csv (e.g. Figure 3b use-case matrix) loaded into the containerized SMetaS application
- SMetaS fixed vocabularies (zeroTimeEvent, drugName, drugDoseUnit, etc.) after the n-grams/nearest-neighbors and substring-search passes
- List of residual unmapped non-numeric strings flagged for user confirmation

## Outputs

- User-confirmed new vocabulary terms (e.g. 'allergen exposure' → zeroTimeEvent, 'ozone' → drugName, 'hours/day' → drugDoseUnit)
- Standardized metadata csv (standardized_metadata.csv, corresponding to Figure 3c) with all non-numeric strings mapped to vocabulary terms

## How to apply

First load the use-case freetext metadata csv (e.g. the Figure 3b ozone/house dust mite lung metabolomics matrix) into the containerized SMetaS application, which replaces freetext with fixed-vocabulary terms using synonym matching and typographical fixing in an n-grams/nearest-neighbors (tf-idf/nnm) model approach. Run the substring-search pass over the fixed vocabularies to resolve any remaining unmapped non-numeric strings. Then, for strings still unmapped after both automated passes, invoke the user-confirmation step: in the ozone/lung use case exactly three terms were added — 'allergen exposure' to the zeroTimeEvent vocabulary, 'ozone' to the drugName vocabulary, and 'hours/day' to the drugDoseUnit vocabulary. Each new term must be assigned to the correct target vocabulary rather than appended globally. Finally export the standardized table (Figure 3c) as standardized_metadata.csv and validate that every non-numeric string is now mapped with no freetext remnants.

## Related tools

- **SMetaS** (Containerized application (database + API + frontend) in which the two automated mapping passes run and whose user-confirmation step is used to add the new terms to the fixed vocabularies before exporting the standardized csv) — https://github.com/metabolomics-us/metadatastandardizer
- **MetaS** (The underlying MetaS/MetaS tool whose extensive documentation covers the metadata standardization workflow executed here) — https://metabolomics-us.github.io/metadatastandardizer/

## Evaluation signals

- All non-numeric strings in the exported standardized_metadata.csv are mapped to vocabulary terms — no freetext remnants remain
- Exactly the confirmed new terms appear under the correct vocabularies: 'allergen exposure' under zeroTimeEvent, 'ozone' under drugName, 'hours/day' under drugDoseUnit
- Diff of the input freetext csv (Figure 3b) against the standardized csv (Figure 3c) shows changes only for the unmapped strings plus their confirmed new-term replacements
- No strings were added to vocabularies without passing through the user-confirmation step (i.e. the three new terms match the user-approved list exactly)

## Limitations

- Only strings that remain unmapped after both the n-grams/nearest-neighbors pass and the substring-search pass are eligible for new-term addition; strings resolvable by synonym matching or typographical fixing should not trigger this step
- New-term addition is inherently manual and user-confirmed, so it does not scale to large volumes of unmapped metadata without human review
- The demonstration AWS instance is slow because of low i/o speed, though the project can run much faster when self-hosted

## Evidence

- [abstract] n-grams/nearest neighbors mapping pass grounds the pre-confirmation passes: "This transformation process is designed to maximize simplicity and is guided by, among other strategies, synonym matching and typographical fixing in an n-grams/nearest neighbors model approach."
- [abstract] containerized SMetaS platform in which the confirmation step is executed: "SMetaS (Sample Metadata Standardizer) combines a classic database with an API and frontend and is provided in a containerized environment."
- [readme] documentation entry point for the MetaS workflow: "Extensive documentation is available at https://metabolomics-us.github.io/metadatastandardizer/"
