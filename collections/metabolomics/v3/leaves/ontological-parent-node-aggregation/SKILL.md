---
name: ontological-parent-node-aggregation
description: Use when SMetaS-standardized sample metadata must be compared across
  studies at their highest ontological parent nodes, or when validating large-scale
  aggregated metadata queries against known biological ground truths.
when_to_use_negative:
- When the goal is standardizing raw freetext metadata itself rather than aggregating
  already-standardized terms — that upstream step belongs to SMetaS's synonym-matching/typo-fixing
  transformation, not this aggregation skill
- When studies must be compared at fine-grained (leaf-level) term granularity rather
  than at their highest ontological parent nodes
- When no published ground truths are available to validate the aggregated queries
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3071
- http://edamontology.org/topic_3489
tools:
- name: SMetaS
  role: Produces the standardized metadata (fixed-vocabulary terms) that this skill
    aggregates by highest ontological parent node; provided as a containerized environment
    with database, API, and frontend
  repo: github.com/metabolomics-us/metadatastandardizer
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.3390/metabo13080941
    title: 'SMetaS: A Sample Metadata Standardizer for Metabolomics'
  source_content_hash: sha256:f3f135c2f5d8e46f565591a83e67f4a025c37d14c4018a7d8b84b8792ae36e86
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - SMetaS
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3071
  - http://edamontology.org/topic_3489
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# ontological-parent-node-aggregation

## Summary

Aggregates SMetaS-standardized sample metadata at the highest ontological parent nodes of a reference vocabulary to build kingdom-/domain-level summary tables, enabling large-scale cross-database queries that can be validated against published ground truths (e.g., absence of cholesterol in the plant kingdom).

## When to use

Use when SMetaS-standardized sample metadata must be compared across studies at their highest ontological parent nodes, or when validating large-scale aggregated metadata queries against known biological ground truths.

## When NOT to use

- When the goal is standardizing raw freetext metadata itself rather than aggregating already-standardized terms — that upstream step belongs to SMetaS's synonym-matching/typo-fixing transformation, not this aggregation skill
- When studies must be compared at fine-grained (leaf-level) term granularity rather than at their highest ontological parent nodes
- When no published ground truths are available to validate the aggregated queries

## Inputs

- SMetaS-standardized sample metadata (freetext terms replaced by terms from fixed vocabularies)
- Reference vocabulary/ontology defining highest parent nodes for each standardized term
- List of published ground truths to verify (e.g., absence of cholesterol in the plant kingdom)

## Outputs

- Kingdom-/domain-level summary table aggregating sample counts under highest ontological parent nodes
- ground_truth_verification.csv recording, for each queried ground truth, whether the aggregated result is consistent with known biology

## How to apply

First load SMetaS-standardized sample metadata in which freetext terms have already been replaced by terms from fixed vocabularies (via synonym matching and typographical fixing in an n-grams/nearest neighbors model approach). Map each standardized metadata term to its highest ontological parent node in the reference vocabulary, then aggregate sample counts under these top-level parent nodes to build a kingdom-/domain-level summary table. Query the aggregated table for entries matching each ground truth to be verified — for example, samples of the plant kingdom annotated with cholesterol. For each queried ground truth, record whether the aggregated result is consistent with known biology, writing the per-query verdicts to ground_truth_verification.csv. Accept the output only if the aggregated queries reproduce the known ground truths (e.g., zero plant-kingdom samples annotated with cholesterol), consistent with the paper's goal that standardized metadata enable finding datasets with comparable metadata and supporting data reuse.

## Related tools

- **SMetaS** (Produces the standardized metadata (fixed-vocabulary terms) that this skill aggregates by highest ontological parent node; provided as a containerized environment with database, API, and frontend) — github.com/metabolomics-us/metadatastandardizer

## Evaluation signals

- ground_truth_verification.csv contains a verdict for every queried ground truth, and each verdict is 'consistent' when the aggregated query reproduces known biology (e.g., zero plant-kingdom samples annotated with cholesterol)
- Every term in the aggregated table maps to a valid highest ontological parent node in the reference vocabulary (no unmapped standardized terms)
- The aggregated sample counts at top-level parent nodes sum to the total number of samples in the input standardized metadata
- Queries run against the aggregated table, not raw per-sample records — verifying the kingdom-/domain-level summarization step actually occurred
- Downstream dataset discovery succeeds: standardized metadata enable finding datasets with comparable metadata and data reuse, per the paper's stated FAIRness goal

## Limitations

- Validation depends entirely on the completeness and correctness of the published ground truths used; novel or contested biological claims cannot be checked this way
- Aggregation at the highest parent node loses fine-grained term detail, so queries below the kingdom-/domain-level summary are out of scope
- Downstream use assumes SMetaS standardization succeeded; unstandardized freetext terms will not map to parent nodes correctly
- The example hosted MetaS instance is slow due to low i/o speed on the AWS instance, though the project can run much faster

## Evidence

- [abstract] Aggregation at highest parent nodes; validation via published ground truths: "Using such a tool in frontends for metabolomic databases will dramatically increase the FAIRness (Findability, Accessibility, Interoperability, and Reusability) of data"
- [abstract] Standardization mechanism underlying the aggregated terms: "This transformation process is designed to maximize simplicity and is guided by, among other strategies, synonym matching and typographical fixing in an n-grams/nearest neighbors model approach."
- [abstract] Tool availability and containerized delivery: "SMetaS (Sample Metadata Standardizer) combines a classic database with an API and frontend and is provided in a containerized environment."
- [readme] Documentation and hosted example instance: "Extensive documentation is available at https://metabolomics-us.github.io/metadatastandardizer/"
- [readme] Hosted instance performance caveat: "This instance is slow because we chose to pay for low i/o speed on our AWS instance. The project can run much faster."
