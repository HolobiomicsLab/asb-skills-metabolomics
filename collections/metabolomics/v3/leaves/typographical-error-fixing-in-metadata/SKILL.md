---
name: typographical-error-fixing-in-metadata
description: Use when a freetext sample metadata table (e.g. csv) contains non-numeric
  strings like 'allergen exposure' or 'ozone' that must be mapped onto fixed vocabularies
  such as zeroTimeEvent, drugName, or drugDoseUnit.
when_to_use_negative:
- The metadata fields are purely numeric (the passes target non-numeric freetext strings;
  numeric values are not vocabulary-mapped).
- The metadata already conforms to the fixed vocabularies with no freetext remnants
  — re-running adds no value.
- You need to build new vocabularies/ontologies from scratch rather than map existing
  freetext onto them.
edam_operation: http://edamontology.org/operation_0306
edam_topics:
- http://edamontology.org/topic_3318
tools:
- name: SMetaS
  role: Containerized application (database + API + frontend) whose n-grams/nearest-neighbors
    pass performs the synonym matching and typographical fixing of freetext metadata
    terms
  repo: https://github.com/metabolomics-us/metadatastandardizer
- name: MetaS
  role: The documented implementation/distribution of the standardizer whose documentation
    covers running the transformation passes
  repo: https://metabolomics-us.github.io/metadatastandardizer/
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.3390/metabo13080941
    title: 'SMetaS: A Sample Metadata Standardizer for Metabolomics'
  source_content_hash: sha256:c9127b7cda02be5c2243d932a90916eb4803736d1b1b4e7f9c9a09c6d3563739
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/metabolomics-us/metadatastandardizer
  tools:
  - MetaS
  - SMetaS
  edam_operation: http://edamontology.org/operation_0306
  edam_topics:
  - http://edamontology.org/topic_3318
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8235294117647058
n_claims: 17
---

# typographical-error-fixing-in-metadata

## Summary

Applies SMetaS's n-grams/nearest-neighbors (tf-idf/nnm) transformation with synonym matching and typographical fixing to replace freetext sample metadata strings with terms from fixed, ontology-derived vocabularies, escalating only truly unmapped strings to a user-confirmation step.

## When to use

Use when a freetext sample metadata table (e.g. csv) contains non-numeric strings like 'allergen exposure' or 'ozone' that must be mapped onto fixed vocabularies such as zeroTimeEvent, drugName, or drugDoseUnit.

## When NOT to use

- The metadata fields are purely numeric (the passes target non-numeric freetext strings; numeric values are not vocabulary-mapped).
- The metadata already conforms to the fixed vocabularies with no freetext remnants — re-running adds no value.
- You need to build new vocabularies/ontologies from scratch rather than map existing freetext onto them.

## Inputs

- Freetext sample metadata csv (e.g. the Figure 3b use-case matrix for the ozone/house dust mite lung metabolomics study)
- Initial ontology-derived fixed vocabularies (e.g. zeroTimeEvent, drugName, drugDoseUnit)

## Outputs

- standardized_metadata.csv — the standardized metadata table with all non-numeric strings mapped to vocabulary terms (cf. Figure 3c)
- Set of user-confirmed new vocabulary terms added to the affected vocabularies

## How to apply

Load the freetext metadata csv (e.g. the ozone/house dust mite lung metabolomics use-case matrix) into the containerized SMetaS application. Run the first curation pass — the n-grams/nearest-neighbors (tf-idf/nnm) model — which performs synonym matching and typographical fixing to substitute freetext terms with vocabulary terms. Run a second substring-search pass over the fixed vocabularies to catch strings the nnm pass missed. Only for strings left unmapped by both passes, invoke the user-confirmation step to add new terms (in the ozone use case exactly three: 'allergen exposure' to zeroTimeEvent, 'ozone' to drugName, 'hours/day' to drugDoseUnit). Export the standardized table as a csv; the rationale is maximizing simplicity of the transformation while boosting FAIRness of the metadata.

## Related tools

- **SMetaS** (Containerized application (database + API + frontend) whose n-grams/nearest-neighbors pass performs the synonym matching and typographical fixing of freetext metadata terms) — https://github.com/metabolomics-us/metadatastandardizer
- **MetaS** (The documented implementation/distribution of the standardizer whose documentation covers running the transformation passes) — https://metabolomics-us.github.io/metadatastandardizer/

## Evaluation signals

- All non-numeric strings in the exported standardized csv are mapped — no freetext remnants remain.
- Exactly the user-confirmed new terms appear, and only under the correct vocabularies (in the reference use case: 'allergen exposure' under zeroTimeEvent, 'ozone' under drugName, 'hours/day' under drugDoseUnit).
- Terms resolved by passes 1–2 must come from the initial ontology-derived vocabularies, not be silently invented by the model.
- The standardized csv matches the expected Figure 3c-style output structure for the use case.

## Limitations

- Strings not resolved by the tf-idf/nnm or substring-search passes require manual user confirmation before being added as new vocabulary terms — the passes alone do not guarantee full coverage.
- The hosted example instance is slow because of low i/o speed on the AWS instance; a locally run container is faster.
- No changelog is available for tracking tool version changes (discussion notes 'No changelog found').

## Evidence

- [abstract] Synonym matching and typographical fixing in n-grams/nearest-neighbors model: "This transformation process is designed to maximize simplicity and is guided by, among other strategies, synonym matching and typographical fixing in an n-grams/nearest neighbors model approach."
- [abstract] SMetaS containerized environment: "SMetaS (Sample Metadata Standardizer) combines a classic database with an API and frontend and is provided in a containerized environment."
- [results] Use-case outcome: three new terms added: [evidence span withheld — anchor status: not_found]
- [abstract] FAIRness rationale: "Using such a tool in frontends for metabolomic databases will dramatically increase the FAIRness (Findability, Accessibility, Interoperability, and Reusability) of data"
- [readme] Documentation location: "Extensive documentation is available at https://metabolomics-us.github.io/metadatastandardizer/"
