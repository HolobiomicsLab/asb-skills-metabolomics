---
name: chemistry-dashboard-metadata-joining
description: Use when a compiled chemical list (e.g. CASRN-indexed exposome entries)
  needs chemical names, synonyms, and structural identifiers such as canonical SMILES
  and InChIKey attached from the U.S. EPA Chemistry Dashboard.
when_to_use_negative:
- The chemical list lacks CASRN identifiers — the U.S. EPA Chemistry Dashboard only
  retrieves domain data for chemicals with CASRN, so metadata joining will fail or
  silently drop entries
- The task is predicting biotransformation metabolites or half-lives (use BioTransformer
  or the Iterative Fragment Selection approach instead) rather than enriching a parent-chemical
  list with identifiers
edam_operation: http://edamontology.org/operation_0224
edam_topics: []
tools:
- name: U.S. EPA Chemistry Dashboard
  role: External lookup service joined on CASRN to attach chemical names, synonyms,
    and structural identifiers (SMILES/InChIKey) to the compiled exposome chemical
    set
- name: HExpMetDB
  role: Resulting human exposome database resource into which the CASRN-keyed, dashboard-enriched
    chemical metadata is compiled and served
  repo: github.com/FangLabNTU/HExpMetDB
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1289/ehp7722
    title: Risk-Based Chemical Ranking and Generating a Prioritized Human Exposome
      Database
  source_content_hash: sha256:17ae2f0b20f12d7fd9b22dfb3d84ddceaa9940c65394235a8079704196b029a7
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - HExpMetDB
  - U.S. EPA Chemistry Dashboard
  edam_operation: http://edamontology.org/operation_0224
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# Chemistry Dashboard Metadata Joining

## Summary

A database-fusion skill for attaching chemical names, synonyms, and structural identifiers (SMILES/InChIKey) to a compiled chemical list by joining against the U.S. EPA Chemistry Dashboard keyed on CASRN, as used to build the >20,000-chemical human exposome database in the risk-based chemical ranking workflow.

## When to use

Use when a compiled chemical list (e.g. CASRN-indexed exposome entries) needs chemical names, synonyms, and structural identifiers such as canonical SMILES and InChIKey attached from the U.S. EPA Chemistry Dashboard.

## When NOT to use

- The chemical list lacks CASRN identifiers — the U.S. EPA Chemistry Dashboard only retrieves domain data for chemicals with CASRN, so metadata joining will fail or silently drop entries
- The task is predicting biotransformation metabolites or half-lives (use BioTransformer or the Iterative Fragment Selection approach instead) rather than enriching a parent-chemical list with identifiers

## Inputs

- Table of chemical entries fused from source databases (EPA HPV List, EINECS, EPA CHEMINV, NIH Tox21, EPA pesticide lists, Exposome-Explorer, Blood Exposome, dust exposome), each with CASRN where available
- CASRN list of chemicals remaining after removing inorganic compounds, organic mixtures, and metals and deduplicating by CASRN and canonical SMILES

## Outputs

- Non-redundant human exposome chemical table keyed by unique CASRN (20,756 unique chemicals in the reference study)
- Per-chemical metadata: chemical names, synonyms, and structural identifiers (canonical SMILES and InChIKey) retrieved from the U.S. EPA Chemistry Dashboard

## How to apply

First fuse chemical entries from source databases (EPA HPV List, EINECS, EPA CHEMINV, NIH Tox21, EPA pesticides, Exposome-Explorer, Blood Exposome, dust exposome) into a single table keyed by chemical identity, then filter out inorganic compounds, organic mixtures, and metals, and deduplicate by CASRN and canonical SMILES to obtain the unique chemical set. Next, join chemical names, synonyms, and structural identifiers (SMILES/InChIKey) from the U.S. EPA Chemistry Dashboard using CASRN as the join key. Note that the dashboard's high-throughput retrieval only succeeds for chemicals that carry a CASRN, so entries lacking a CASRN will drop out at this step. Validate the join by confirming the compiled list still contains the expected number of unique CASRN chemicals (20,756 in the paper) and that dominant contributors (EPA HPV, EINECS, CHEMINV, Tox21, EPA pesticides) cover 93% (18,909/20,756) of the database.

## Related tools

- **U.S. EPA Chemistry Dashboard** (External lookup service joined on CASRN to attach chemical names, synonyms, and structural identifiers (SMILES/InChIKey) to the compiled exposome chemical set)
- **HExpMetDB** (Resulting human exposome database resource into which the CASRN-keyed, dashboard-enriched chemical metadata is compiled and served) — github.com/FangLabNTU/HExpMetDB

## Evaluation signals

- Compiled, deduplicated list contains 20,756 unique CASRN chemicals (matching the reference exposome database size)
- Dominant source databases (EPA HPV, EINECS, CHEMINV, Tox21, EPA pesticides) together cover 93% (18,909/20,756) of the compiled list
- Every retained chemical is keyed by a unique CASRN and has been deduplicated by canonical SMILES
- Retrieved metadata rows carry non-empty chemical name, synonym set, and SMILES/InChIKey structural identifiers joined from the Chemistry Dashboard
- No inorganic compounds, organic mixtures, or metals remain in the joined table

## Limitations

- Only chemicals with CASRN successfully retrieve diverse types of relevant domain data from the U.S. EPA Chemistry Dashboard; chemicals without CASRN cannot be enriched
- The metadata join operates on parent chemicals only; the study notes the need for periodic updates to incorporate new chemicals in the future
- Coverage of the joined list depends on upstream filtering (inorganics, mixtures, metals removed) and on the SEEM and other data sources being unable to cover all compounds

## Evidence

- [discussion] our HTS method is based on the U.S. EPA Chemistry Dashboard, only chemicals with CASRN were successful in retrieving diverse types of relevant domain data: "our HTS method is based on the U.S. EPA Chemistry Dashboard, only chemicals with CASRN were successful in retrieving diverse types of relevant domain data"
- [methods] Deduplicate by CASRN and canonical SMILES to obtain the unique chemical set.: [evidence span withheld — anchor status: not_found]
- [methods] Join chemical names, synonyms, and structural identifiers (SMILES/InChIKey) from the EPA Chemistry Dashboard.: [evidence span withheld — anchor status: not_found]
