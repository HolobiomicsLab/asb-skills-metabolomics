---
name: exposome-database-compilation
description: Use when the goal is to build a large, deduplicated human exposome chemical
  inventory from multiple public databases and literature sources, keyed by CASRN
  with SMILES/InChIKey annotations.
when_to_use_negative:
- The input is already a small, curated chemical list for a targeted analysis — database
  fusion and deduplication are unnecessary
- The downstream goal is toxicity prediction, exposure estimation, or risk-based prioritization
  of an existing chemical set (SEEM, TEST, ToxPi, ToxCast steps) rather than building
  the inventory itself
- Chemicals of interest lack CASRN and cannot be resolved through the U.S. EPA Chemistry
  Dashboard, so the annotation join will fail
edam_operation: http://edamontology.org/operation_2422
tools:
- name: U.S. EPA Chemistry Dashboard
  role: Joins chemical names, synonyms, and structural identifiers (SMILES/InChIKey)
    onto the deduplicated CASRN-keyed chemical set; retrieval of domain data only
    succeeds for chemicals with CASRN
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1289/ehp7722
    title: Risk-Based Chemical Ranking and Generating a Prioritized Human Exposome
      Database
  source_content_hash: sha256:cfea677500033a454c67c83eae28439f596742d46433173f85dbd5ad8034b8bd
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - U.S. EPA Chemistry Dashboard
  edam_operation: http://edamontology.org/operation_2422
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 5
---

# exposome-database-compilation

## Summary

Compiles a comprehensive human exposome chemical inventory (>20,000 chemicals) by text mining and fusing public databases (EPA HPV, EINECS, CHEMINV, NIH Tox21, EPA pesticides, Exposome-Explorer, Blood Exposome, dust exposome), then filtering and deduplicating to a unique CASRN-keyed list with structural identifiers.

## When to use

Use when the goal is to build a large, deduplicated human exposome chemical inventory from multiple public databases and literature sources, keyed by CASRN with SMILES/InChIKey annotations.

## When NOT to use

- The input is already a small, curated chemical list for a targeted analysis — database fusion and deduplication are unnecessary
- The downstream goal is toxicity prediction, exposure estimation, or risk-based prioritization of an existing chemical set (SEEM, TEST, ToxPi, ToxCast steps) rather than building the inventory itself
- Chemicals of interest lack CASRN and cannot be resolved through the U.S. EPA Chemistry Dashboard, so the annotation join will fail

## Inputs

- Chemical entry lists from source databases: EPA HPV List, EINECS, EPA CHEMINV, NIH Tox21, EPA pesticide lists
- Chemical entries from exposome-specific resources: Exposome-Explorer, Blood Exposome Database, dust exposome database
- Text-mined chemical mentions from literature sources

## Outputs

- Deduplicated human exposome chemical table of 20,756 unique CASRN chemicals
- Per-chemical annotations: chemical names, synonyms, CASRN, canonical SMILES, InChIKey (joined from the U.S. EPA Chemistry Dashboard)
- Per-source contribution counts showing dominant database coverage (18,909/20,756 = 93%)

## How to apply

Collect chemical entries from the source databases cited in the Methods (EPA HPV List, EINECS, EPA CHEMINV, NIH Tox21, EPA pesticide lists, Exposome-Explorer, Blood Exposome, dust exposome, and text-mined literature sources). Merge all entries into a single table keyed by chemical identity, then filter out inorganic compounds, organic mixtures, and metals. Deduplicate by CASRN and canonical SMILES to obtain the unique chemical set, and join chemical names, synonyms, and structural identifiers (SMILES/InChIKey) from the U.S. EPA Chemistry Dashboard. Rationale: fusion of broad inventory databases dominates coverage — EPA HPV, EINECS, CHEMINV, Tox21, and EPA pesticide lists together account for 93% (18,909/20,756) of the compiled database — while exposome-specific databases add measured-exposure chemicals absent from inventories. Note that retrieval of domain data via the Chemistry Dashboard only succeeds for chemicals with a CASRN, so CASRN resolution is the critical identity step.

## Related tools

- **U.S. EPA Chemistry Dashboard** (Joins chemical names, synonyms, and structural identifiers (SMILES/InChIKey) onto the deduplicated CASRN-keyed chemical set; retrieval of domain data only succeeds for chemicals with CASRN)

## Evaluation signals

- Final deduplicated table contains exactly 20,756 unique CASRN chemicals
- Dominant source databases (EPA HPV, EINECS, CHEMINV, Tox21, EPA pesticides) collectively cover 93% of entries (18,909/20,756)
- No inorganic compounds, organic mixtures, or metals remain after filtering
- No duplicate CASRN or canonical SMILES values remain in the merged table
- Compiled list size is consistent with the article's reported '>20,000 chemicals' human exposome database

## Limitations

- Only chemicals with CASRN successfully retrieve diverse types of relevant domain data from the U.S. EPA Chemistry Dashboard
- The compiled database should be periodically updated to incorporate new chemicals in the future
- The fusion scope excludes inorganic compounds, organic mixtures, and metals by design

## Evidence

- [abstract] We built a comprehensive risk-prioritized human exposome database by text mining and database fusion: "We built a comprehensive risk-prioritized human exposome database by text mining and database fusion"
- [abstract] We compiled a human exposome database of >20,000 chemicals: "We compiled a human exposome database of >20,000 chemicals"
- [other] Filter out inorganic compounds, organic mixtures, and metals: [evidence span withheld — anchor status: not_found]
- [other] Deduplicate by CASRN and canonical SMILES: [evidence span withheld — anchor status: not_found]
- [discussion] CASRN-only Dashboard retrieval: "only chemicals with CASRN were successful in retrieving diverse types of relevant domain data from the U.S. EPA Chemistry Dashboard"
- [other] 20,756 unique CASRN chemicals dominated by five inventories: [evidence span withheld — anchor status: not_found]
