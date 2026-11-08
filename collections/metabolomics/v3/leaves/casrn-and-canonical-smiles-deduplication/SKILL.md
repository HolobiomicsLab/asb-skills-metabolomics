---
name: casrn-and-canonical-smiles-deduplication
description: Use when fusing multiple chemical source databases or lists into one
  exposome/inventory table and you must eliminate duplicates and non-applicable chemical
  classes before downstream prioritization or annotation.
when_to_use_negative:
- The target set includes inorganic compounds, metals, or organic mixtures, which
  are explicitly filtered out of this exposome database
- The chemicals lack CASRN — the paper notes only chemicals with CASRN could retrieve
  diverse domain data from the U.S. EPA Chemistry Dashboard
- The task is metabolite prediction (BioTransformer) or toxicity estimation (TEST/ToxPi)
  rather than building the parent-chemical master list
edam_operation: http://edamontology.org/operation_2422
edam_topics:
- http://edamontology.org/topic_2258
- http://edamontology.org/topic_3314
tools:
- name: U.S. EPA Chemistry Dashboard
  role: Source of chemical names, synonyms, and structural identifiers (SMILES/InChIKey)
    joined onto the deduplicated CASRN set
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1289/ehp7722
    title: Risk-Based Chemical Ranking and Generating a Prioritized Human Exposome
      Database
  source_content_hash: sha256:859ae03e55ef6a4e46ab2018e1716e6495cd4f804ccaaefa718bfcbfdea80eec
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - U.S. EPA Chemistry Dashboard
  edam_operation: http://edamontology.org/operation_2422
  edam_topics:
  - http://edamontology.org/topic_2258
  - http://edamontology.org/topic_3314
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# CASRN and canonical SMILES deduplication

## Summary

A database-fusion deduplication step that merges chemical entries from multiple exposome and inventory databases (EPA HPV, EINECS, EPA CHEMINV, NIH Tox21, EPA pesticides, Exposome-Explorer, Blood Exposome, dust exposome) into a single table keyed by chemical identity, then removes inorganic compounds, organic mixtures, and metals and deduplicates by CASRN and canonical SMILES to yield the unique human exposome chemical set.

## When to use

Use when fusing multiple chemical source databases or lists into one exposome/inventory table and you must eliminate duplicates and non-applicable chemical classes before downstream prioritization or annotation.

## When NOT to use

- The target set includes inorganic compounds, metals, or organic mixtures, which are explicitly filtered out of this exposome database
- The chemicals lack CASRN — the paper notes only chemicals with CASRN could retrieve diverse domain data from the U.S. EPA Chemistry Dashboard
- The task is metabolite prediction (BioTransformer) or toxicity estimation (TEST/ToxPi) rather than building the parent-chemical master list

## Inputs

- Chemical entry tables from source databases (EPA HPV List, EINECS, EPA CHEMINV, NIH Tox21, EPA pesticides, Exposome-Explorer, Blood Exposome, dust exposome)
- Chemical names and synonyms from each source
- Structural identifiers (CASRN, canonical SMILES, InChIKey) retrievable from the U.S. EPA Chemistry Dashboard

## Outputs

- Deduplicated human exposome chemical table keyed by unique CASRN (expected size: 20,756 chemicals)
- Chemical annotation fields: names, synonyms, canonical SMILES/InChIKey from the EPA Chemistry Dashboard
- Per-source contribution counts (e.g., dominant sources covering 18,909/20,756 = 93%)

## How to apply

Collect chemical entries from each source database cited in the Methods (EPA HPV List, EINECS, EPA CHEMINV, NIH Tox21, EPA pesticides, Exposome-Explorer, Blood Exposome, dust exposome). Merge the entries into a single table keyed by chemical identity, then filter out inorganic compounds, organic mixtures, and metals, since these classes fall outside the human exposome scope of the paper. Deduplicate by CASRN and canonical SMILES so each chemical appears once in the final set. Join chemical names, synonyms, and structural identifiers (SMILES/InChIKey) from the U.S. EPA Chemistry Dashboard, using CASRN as the retrieval key. Validate by comparing the compiled list against the paper's reported results: 20,756 unique CASRN chemicals, with the five dominant sources (EPA HPV, EINECS, CHEMINV, Tox21, EPA pesticides) covering 18,909/20,756 (93%).

## Related tools

- **U.S. EPA Chemistry Dashboard** (Source of chemical names, synonyms, and structural identifiers (SMILES/InChIKey) joined onto the deduplicated CASRN set)

## Evaluation signals

- Final deduplicated table contains exactly 20,756 unique CASRN chemicals
- Dominant source databases (EPA HPV, EINECS, CHEMINV, Tox21, EPA pesticides) together cover 93% (18,909/20,756) of the compiled database
- No duplicate CASRN or duplicate canonical SMILES values remain in the output table
- Inorganic compounds, organic mixtures, and metals are absent from the final chemical set
- Every retained chemical carries name, synonym, and SMILES/InChIKey annotation joined from the EPA Chemistry Dashboard

## Limitations

- Only chemicals with CASRN can successfully retrieve diverse types of relevant domain data from the U.S. EPA Chemistry Dashboard, so chemicals without CASRN are effectively unannotatable in this workflow
- The compiled list reflects the source databases available at compilation time; the article notes the database should be periodically updated to incorporate new chemicals
- Filtering rules (inorganic compounds, organic mixtures, metals) remove chemical classes that some exposomics analyses may need to retain

## Evidence

- [other] Deduplicate by CASRN and canonical SMILES to obtain the unique chemical set.: [evidence span withheld — anchor status: not_found]
- [other] Filter inorganics/mixtures/metals and join identifiers: [evidence span withheld — anchor status: not_found]
- [discussion] CASRN limitation for Chemistry Dashboard retrieval: "only chemicals with CASRN were successful in retrieving diverse types of relevant domain data from the U.S. EPA Chemistry Dashboard"
- [abstract] Database built by text mining and fusion: "We built a comprehensive risk-prioritized human exposome database by text mining and database fusion"
