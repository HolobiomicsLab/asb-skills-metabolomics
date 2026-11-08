---
name: database-chemical-list-fusion
description: Use when building a unified chemical inventory from multiple databases
  and literature sources (e.g., exposome source lists) and you need a deduplicated,
  structurally annotated parent-chemical list.
when_to_use_negative:
- The analysis targets biotransformation metabolites rather than parent chemicals
  — metabolite prediction is handled separately with BioTransformer, not by database
  fusion.
- Chemicals of interest are inorganic compounds, metals, or organic mixtures — these
  are explicitly filtered out during the fusion workflow.
- A chemical lacks a CASRN — the U.S. EPA Chemistry Dashboard cannot retrieve diverse
  types of relevant domain data for it, so downstream annotation will fail.
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3334
tools:
- name: U.S. EPA Chemistry Dashboard
  role: Resolves chemical identities and joins names, synonyms, and structural identifiers
    (SMILES/InChIKey) onto the fused chemical list; retrieval requires a CASRN
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1289/ehp7722
    title: Risk-Based Chemical Ranking and Generating a Prioritized Human Exposome
      Database
  source_content_hash: sha256:633e1f0ffe4c40f3efc6323baf29447b955b1e083c7da8d82faf96254050f989
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - U.S. EPA Chemistry Dashboard
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3334
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# database-chemical-list-fusion

## Summary

Fuses chemical entries from multiple public databases and literature sources into a single deduplicated human exposome chemical inventory keyed by CASRN, filtering out inorganic compounds, organic mixtures, and metals. This produced the >20,000-chemical (20,756 unique CASRN) list underlying the prioritized HExpMetDB resource.

## When to use

Use when building a unified chemical inventory from multiple databases and literature sources (e.g., exposome source lists) and you need a deduplicated, structurally annotated parent-chemical list.

## When NOT to use

- The analysis targets biotransformation metabolites rather than parent chemicals — metabolite prediction is handled separately with BioTransformer, not by database fusion.
- Chemicals of interest are inorganic compounds, metals, or organic mixtures — these are explicitly filtered out during the fusion workflow.
- A chemical lacks a CASRN — the U.S. EPA Chemistry Dashboard cannot retrieve diverse types of relevant domain data for it, so downstream annotation will fail.

## Inputs

- Chemical entries from EPA HPV List
- Chemical entries from EINECS
- Chemical entries from EPA CHEMINV
- Chemical entries from NIH Tox21
- EPA pesticide chemical lists
- Exposome-Explorer Database chemicals (691 chemicals)
- Blood Exposome Database chemicals
- dust exposome database chemicals (511 chemicals with measured concentrations)
- text-mined literature chemical entries

## Outputs

- Deduplicated human exposome chemical list keyed by CASRN (target: 20,756 unique chemicals)
- Table with chemical names, synonyms, and structural identifiers (SMILES/InChIKey) joined from the U.S. EPA Chemistry Dashboard
- Per-source contribution counts (e.g., dominant five databases covering 18,909/20,756 = 93%)

## How to apply

Collect chemical entries from the source databases cited in the Methods — EPA HPV List, EINECS, EPA CHEMINV, NIH Tox21, EPA pesticide lists, Exposome-Explorer, Blood Exposome, and dust exposome — and merge them into a single table keyed by chemical identity. Filter out inorganic compounds, organic mixtures, and metals to restrict the inventory to exposome-relevant parent organic chemicals. Deduplicate by CASRN and canonical SMILES so each unique chemical appears once even when contributed by several sources. Join chemical names, synonyms, and structural identifiers (SMILES/InChIKey) from the U.S. EPA Chemistry Dashboard; note that Dashboard retrieval succeeds only for chemicals that have a CASRN. Validate the result by checking the unique CASRN count against the reported 20,756 and confirming that the dominant contributors (EPA HPV, EINECS, CHEMINV, Tox21, EPA pesticides) cover 93% (18,909/20,756) of the database.

## Related tools

- **U.S. EPA Chemistry Dashboard** (Resolves chemical identities and joins names, synonyms, and structural identifiers (SMILES/InChIKey) onto the fused chemical list; retrieval requires a CASRN)

## Evaluation signals

- The fused database contains exactly 20,756 unique CASRN chemicals.
- The five dominant source databases (EPA HPV, EINECS, CHEMINV, Tox21, EPA pesticides) jointly cover 93% of entries (18,909/20,756).
- No inorganic compounds, organic mixtures, or metals remain after filtering.
- No duplicate entries survive deduplication by CASRN and canonical SMILES.
- Every retained chemical carries joined identifiers (name, synonyms, SMILES/InChIKey) from the EPA Chemistry Dashboard.

## Limitations

- The U.S. EPA Chemistry Dashboard-based retrieval only succeeds for chemicals with a CASRN, so chemicals without CASRN drop out of downstream domain-data annotation.
- Filtering deliberately excludes inorganic compounds, organic mixtures, and metals, so the fused list is not a universal chemical inventory.
- The database should be periodically updated to incorporate new chemicals in the future, as the fused snapshot ages.

## Evidence

- [abstract] We built a comprehensive risk-prioritized human exposome database by text mining and database fusion: "We built a comprehensive risk-prioritized human exposome database by text mining and database fusion"
- [methods] Filter out inorganic compounds, organic mixtures, and metals.: [evidence span withheld — anchor status: not_found]
- [methods] Deduplicate by CASRN and canonical SMILES to obtain the unique chemical set.: [evidence span withheld — anchor status: not_found]
- [discussion] only chemicals with CASRN were successful in retrieving diverse types of relevant domain data from the U.S. EPA Chemistry Dashboard: "only chemicals with CASRN were successful in retrieving diverse types of relevant domain data from the U.S. EPA Chemistry Dashboard"
- [abstract] We compiled a human exposome database of >20,000 chemicals: "We compiled a human exposome database of >20,000 chemicals"
