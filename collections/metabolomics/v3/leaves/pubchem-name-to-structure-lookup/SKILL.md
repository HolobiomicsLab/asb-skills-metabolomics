---
name: pubchem-name-to-structure-lookup
description: Use when a metadata sheet for a plate-based small-molecule collection
  lacks a structure field (SMILES/InChI) for some rows but retains compound names,
  and standardized structures are needed downstream.
when_to_use_negative:
- When the input table already carries valid structure information for every row —
  the PubChem Name-search fallback is only triggered if no structure information is
  provided.
- When no compound name is available for a structure-less row — the Name search cannot
  resolve structures for entries without a usable name.
edam_operation: http://edamontology.org/operation_3561
edam_topics:
- http://edamontology.org/topic_3336
- http://edamontology.org/topic_0154
tools:
- name: PubChem
  role: Name-search database used to retrieve structures for compounds whose metadata
    lacks structure information
- name: Prefect
  role: Orchestrates the metadata cleanup flow in which the PubChem name-to-structure
    fallback is executed
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41592-025-02813-0
    title: 'MSnLib: efficient generation of open multi-stage fragmentation mass spectral
      libraries'
  source_content_hash: sha256:27dc3a186778ffca8b9b2f40b24e80405b7a4c33d05ec4b164f9d6a4e9a2673e
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Broad Institute Drug Repurposing Hub
  - Dictionary of Natural Products
  - DrugBank
  - DrugCentral
  - LOTUS
  - Prefect
  - PubChem
  - drugbank_extraction.py
  - pip
  edam_operation: http://edamontology.org/operation_3561
  edam_topics:
  - http://edamontology.org/topic_3336
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# pubchem-name-to-structure-lookup

## Summary

Backfill missing compound structure information in a metadata cleanup pipeline by querying PubChem with the compound Name search, so structure-dependent standardization and descriptor computation can proceed for every sample row.

## When to use

Use when a metadata sheet for a plate-based small-molecule collection lacks a structure field (SMILES/InChI) for some rows but retains compound names, and standardized structures are needed downstream.

## When NOT to use

- When the input table already carries valid structure information for every row — the PubChem Name-search fallback is only triggered if no structure information is provided.
- When no compound name is available for a structure-less row — the Name search cannot resolve structures for entries without a usable name.

## Inputs

- harmonized metadata table (template columns: unique sample id, plate_id, well_location, compound name, structure fields) with rows lacking a structure entry but carrying a compound name

## Outputs

- structures backfilled for previously structure-less rows (raw structure strings suitable for downstream ChEMBL standardization)
- cleaned/standardized metadata table with structure descriptor fields (SMILES, InChI, InChIKey, logP, monoisotopic mass) populated for all rows

## How to apply

During the metadata cleanup flow, detect rows in the harmonized metadata table that have no structure information (no SMILES, InChI, or InChIKey). For each such row, use the compound name as the query term in a PubChem Name search to retrieve the corresponding structure. PubChem is the default fallback; other databases (ChEMBL, Dictionary of Natural Products, LOTUS, Broad Drug Repurposing Hub, DrugBank, DrugCentral) are optional extras, and databases that need a local file or special access should be set to False in jobs.py. The retrieved structure is then fed into the ChEMBL standardization pipeline (salt stripping, neutralization, canonicalization) and descriptor computation (canonical SMILES, isomeric SMILES, InChI, InChIKey, logP, monoisotopic mass).

## Related tools

- **PubChem** (Name-search database used to retrieve structures for compounds whose metadata lacks structure information)
- **Prefect** (Orchestrates the metadata cleanup flow in which the PubChem name-to-structure fallback is executed)

## Examples

```
Start serving the flow locally by running `metadata_cleanup_prefect.py`, the service will then run and wait for jobs to be submitted (jobs.py)
```

## Evaluation signals

- After the lookup, every input row in the cleaned metadata table has non-empty structure descriptor fields (SMILES, InChI, InChIKey, logP, monoisotopic mass).
- Row count of the output table matches the input metadata sheet, with no rows dropped or duplicated during structure backfill.
- Unique sample ids are non-null for all rows and plate_id / well_location context is preserved.
- Structures retrieved from PubChem pass through the ChEMBL standardization pipeline and produce consistent canonical SMILES/InChIKey values.

## Limitations

- The lookup depends on the compound name being resolvable in PubChem; ambiguous, misspelled, or proprietary names may yield no structure or a wrong match.
- Some other database sources (DrugBank, Dictionary of Natural Products) require a local file and/or special access and must be disabled in jobs.py if unavailable.
- LOTUS data must be refreshed with prepare_wikidata_lotus_prefect.py or an outdated provided file will be used instead.

## Evidence

- [intro] PubChem is the default fallback when no structure information is provided: "If no structure information is provided, it is queried from PubChem by Name search."
- [readme] Optional databases needing local access should be disabled in jobs.py: "For querying other databases, some need a local file and/or special access otherwise set it to False in the jobs.py:"
- [readme] Metadata should follow a shared template with the same column names and minimum needed information: "Please use the [template]...for your metadata for having same column names and minimum needed information for the query."
