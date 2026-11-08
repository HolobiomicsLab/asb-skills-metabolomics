---
name: inorganic-and-mixture-compound-filtering
description: Use when a merged multi-source chemical inventory (e.g., fused EPA HPV,
  EINECS, CHEMINV, Tox21, pesticide, and exposome lists) must be cleaned of non-single-organic
  entries and deduplicated into a unique CASRN-keyed set.
when_to_use_negative:
- The inventory already contains only single organic compounds with unique CASRN keys
  (no dedup or class filtering needed)
- The downstream analysis targets metals or inorganic species explicitly (e.g., IFS
  HLB prediction is anyway inapplicable to metal-containing chemicals, but a metals-focused
  study would not apply this filter)
- Records lack CASRN entirely and cannot be reconciled via the U.S. EPA Chemistry
  Dashboard, since downstream HTS data retrieval requires CASRN
edam_operation: http://edamontology.org/operation_3661
edam_topics:
- http://edamontology.org/topic_2258
tools:
- name: U.S. EPA Chemistry Dashboard
  role: Supplies chemical names, synonyms, and structural identifiers (SMILES/InChIKey)
    joined onto the filtered, deduplicated chemical set
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1289/ehp7722
    title: Risk-Based Chemical Ranking and Generating a Prioritized Human Exposome
      Database
  source_content_hash: sha256:4780c4cf84b47db540b8a76fd23351d861f9d37336f09cbaa5c6163e951d720e
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - U.S. EPA Chemistry Dashboard
  edam_operation: http://edamontology.org/operation_3661
  edam_topics:
  - http://edamontology.org/topic_2258
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 5
---

# inorganic-and-mixture-compound-filtering

## Summary

A curation step in the HExpMetDB database-fusion workflow that removes inorganic compounds, organic mixtures, and metals from merged chemical lists, then deduplicates the remaining entries by CASRN and canonical SMILES to yield a unique chemical inventory.

## When to use

Use when a merged multi-source chemical inventory (e.g., fused EPA HPV, EINECS, CHEMINV, Tox21, pesticide, and exposome lists) must be cleaned of non-single-organic entries and deduplicated into a unique CASRN-keyed set.

## When NOT to use

- The inventory already contains only single organic compounds with unique CASRN keys (no dedup or class filtering needed)
- The downstream analysis targets metals or inorganic species explicitly (e.g., IFS HLB prediction is anyway inapplicable to metal-containing chemicals, but a metals-focused study would not apply this filter)
- Records lack CASRN entirely and cannot be reconciled via the U.S. EPA Chemistry Dashboard, since downstream HTS data retrieval requires CASRN

## Inputs

- Merged multi-database chemical table (EPA HPV, EINECS, EPA CHEMINV, NIH Tox21, EPA pesticides, Exposome-Explorer, Blood Exposome, dust exposome entries)
- Chemical identity fields: chemical name, CASRN, canonical SMILES/InChIKey
- Structural identifier annotations from the U.S. EPA Chemistry Dashboard

## Outputs

- Deduplicated unique-chemical set keyed by CASRN (target: 20,756 unique CASRN chemicals)
- Cleaned chemical table excluding inorganic compounds, organic mixtures, and metals, with names, synonyms, and SMILES/InChIKey joined

## How to apply

After merging source databases (EPA HPV List, EINECS, EPA CHEMINV, NIH Tox21, EPA pesticides, Exposome-Explorer, Blood Exposome, dust exposome) into a single table keyed by chemical identity, filter out inorganic compounds, organic mixtures, and metals, since these lack the organic single-compound structure required for SMILES-based processing and toxicity prediction. Deduplicate the survivors by CASRN and canonical SMILES so that one physical chemical maps to exactly one record. Join chemical names, synonyms, and structural identifiers (SMILES/InChIKey) from the U.S. EPA Chemistry Dashboard; note that only chemicals with a CASRN can retrieve domain data downstream. Validate by comparing the final list size against the expected 20,756 unique CASRN chemicals, of which 18,909 (93%) trace to the five dominant database contributors.

## Related tools

- **U.S. EPA Chemistry Dashboard** (Supplies chemical names, synonyms, and structural identifiers (SMILES/InChIKey) joined onto the filtered, deduplicated chemical set)

## Examples

```
python -c "import pandas as pd; df = pd.read_csv('merged_exposome_chemicals.csv'); df = df[~df.chemical_class.isin(['inorganic','metal','mixture'])]; df = df.drop_duplicates(subset=['CASRN','canonical_smiles']); df.to_csv('unique_exposome_chemicals.csv', index=False)"
```

## Evaluation signals

- Final unique-chemical count equals 20,756 CASRN-keyed entries
- Dominant source databases (EPA HPV, EINECS, CHEMINV, Tox21, EPA pesticides) account for 18,909/20,756 (93%) of the compiled list
- No inorganic compounds, metals, or organic mixtures remain in the filtered table
- Every retained record maps to exactly one canonical SMILES (no duplicate CASRN/SMILES pairs)
- Retained CASRN values resolve successfully against the U.S. EPA Chemistry Dashboard for name/synonym/InChIKey joins

## Limitations

- Only chemicals with CASRN successfully retrieve diverse domain data from the U.S. EPA Chemistry Dashboard, so CASRN-less entries are effectively excluded from downstream prioritization
- Removing metals/inorganics is consistent with the IFS approach's inability to predict HLBs for chemicals with a metal atom or MW >1,000
- The compiled list should be periodically updated to incorporate new chemicals, so the filtered set is a snapshot

## Evidence

- [methods] Filter out inorganic compounds, organic mixtures, and metals.: [evidence span withheld — anchor status: not_found]
- [methods] Deduplicate by CASRN and canonical SMILES to obtain the unique chemical set.: [evidence span withheld — anchor status: not_found]
- [results] The database-fusion workflow yielded 20,756 unique CASRN chemicals, dominated by EPA HPV, EINECS, CHEMINV, Tox21, and EPA pesticide lists.: [evidence span withheld — anchor status: not_found]
- [discussion] HTS method limited to chemicals with CASRN: "only chemicals with CASRN were successful in retrieving diverse types of relevant domain data from the U.S. EPA Chemistry Dashboard"
- [abstract] We built a comprehensive risk-prioritized human exposome database by text mining and database fusion: "We built a comprehensive risk-prioritized human exposome database by text mining and database fusion"
