---
name: salt-stripping-and-neutralization
description: Use when a compound metadata table (template-format columns with structure
  fields such as SMILES) contains salts, counter-ions or unneutralized molecules that
  must be harmonized before descriptor computation and database lookups.
when_to_use_negative:
- Input structures are already fully standardized single-component molecules with
  no salts or counter-ions — re-running salt stripping adds a needless second pass.
- The goal is fragment-ion / spectral data processing (mzmine, MSn trees) rather than
  compound structure standardization.
- Databases that require local files or special access (DrugBank, Dictionary of Natural
  Products, DrugCentral) are needed but their flags are still set to False in jobs.py
  — resolve access first.
edam_operation: http://edamontology.org/operation_3436
edam_topics:
- http://edamontology.org/topic_0089
tools:
- name: Prefect 2
  role: Orchestrates the cleanup_file flow that runs salt stripping, neutralization
    and descriptor computation as a locally served, deployed pipeline with a concurrency-limited
    work pool.
  repo: corinnabrungs/msn_tree_library
- name: pip
  role: Installs the requirements needed to run the flow locally.
  repo: corinnabrungs/msn_tree_library
- name: prepare_wikidata_lotus_prefect.py
  role: Prepares the LOTUS natural-product database so it can be queried during the
    metadata cleanup alongside standardization.
  repo: corinnabrungs/msn_tree_library
- name: drugbank_extraction.py
  role: Extracts drug structures from the DrugBank download so they enter the standardized
    metadata table.
  repo: corinnabrungs/msn_tree_library
- name: PubChem
  role: Supplies missing structures (by Name search) that then undergo salt stripping
    and neutralization.
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41592-025-02813-0
    title: 'MSnLib: efficient generation of open multi-stage fragmentation mass spectral
      libraries'
  source_content_hash: sha256:f419f233402524496bde61e795d2f0638f7caa291137f405e0f0bd3d771b17d9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/corinnabrungs/msn_tree_library
  tools:
  - Broad Institute Drug Repurposing Hub
  - Dictionary of Natural Products
  - DrugBank
  - DrugCentral
  - LOTUS
  - Prefect 2
  - PubChem
  - drugbank_extraction.py
  - pip
  - prepare_wikidata_lotus_prefect.py
  edam_operation: http://edamontology.org/operation_3436
  edam_topics:
  - http://edamontology.org/topic_0089
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# salt-stripping-and-neutralization

## Summary

Chemical standardization step in the metadata cleanup pipeline: removes salts and counter-ions from compound structures (ChEMBL-style processing) and neutralizes/canonicalizes molecules so that standardized SMILES, InChI, InChIKey and monoisotopic mass descriptors are consistent across the seven compound libraries.

## When to use

Use when a compound metadata table (template-format columns with structure fields such as SMILES) contains salts, counter-ions or unneutralized molecules that must be harmonized before descriptor computation and database lookups.

## When NOT to use

- Input structures are already fully standardized single-component molecules with no salts or counter-ions — re-running salt stripping adds a needless second pass.
- The goal is fragment-ion / spectral data processing (mzmine, MSn trees) rather than compound structure standardization.
- Databases that require local files or special access (DrugBank, Dictionary of Natural Products, DrugCentral) are needed but their flags are still set to False in jobs.py — resolve access first.

## Inputs

- Raw compound metadata sheet in the template format (unique sample id, plate_id, well_location, compound name, structure fields such as SMILES)
- Structure strings retrieved from PubChem by Name search for compounds lacking structure information
- jobs.py configuration (external database flags set to False for databases needing local files or special access)

## Outputs

- Cleaned and standardized metadata table with one row per input compound
- Per-compound structural descriptors: canonical SMILES, isomeric SMILES, InChI, InChIKey, logP, monoisotopic mass
- Unique non-null sample ids with retained plate_id / well_location context

## How to apply

Load the raw metadata sheet and harmonize it to the shared template (same column names, minimum needed information), then remove salt/counter-ion components from structure representations. Run structures through the ChEMBL structure pipeline to standardize them (salt stripping, neutralization, canonicalization); a second pass is required when a salt lacks a dot-separated component. For entries without a structure, query one from PubChem by Name search before standardization. After standardization, compute structural descriptors per compound with RDKit/ChEMBL utilities: canonical SMILES, isomeric SMILES, InChI, InChIKey, logP and monoisotopic mass. The whole cleanup runs as a Prefect 2 flow (cleanup_file deployment) served locally and driven from jobs.py, so salt stripping is a reproducible pipeline stage rather than an ad-hoc script.

## Related tools

- **Prefect 2** (Orchestrates the cleanup_file flow that runs salt stripping, neutralization and descriptor computation as a locally served, deployed pipeline with a concurrency-limited work pool.) — corinnabrungs/msn_tree_library
- **pip** (Installs the requirements needed to run the flow locally.) — corinnabrungs/msn_tree_library
- **prepare_wikidata_lotus_prefect.py** (Prepares the LOTUS natural-product database so it can be queried during the metadata cleanup alongside standardization.) — corinnabrungs/msn_tree_library
- **drugbank_extraction.py** (Extracts drug structures from the DrugBank download so they enter the standardized metadata table.) — corinnabrungs/msn_tree_library
- **PubChem** (Supplies missing structures (by Name search) that then undergo salt stripping and neutralization.)

## Examples

```
prefect deploy metadata_cleanup_prefect:cleanup_file --name local-deploy --pool local-work
```

## Evaluation signals

- Output table exists and contains one row per input compound — row count matches the input metadata sheet.
- Standardized descriptor fields (SMILES, InChI, InChIKey, logP, monoisotopic mass) are populated for every compound.
- Every row carries a unique, non-null sample id.
- Template column names and minimum needed information are preserved in the harmonized output.
- Field presence checks pass after the Prefect flow completes.

## Limitations

- Salts without a dot separator require a second structure-pipeline pass to strip correctly.
- Some external databases need a local file and/or special access and must otherwise be set to False in jobs.py.
- Compounds without any structure field can only be handled if PubChem resolves the compound by Name search.

## Evidence

- [methods] Remove salt/counter-ion components from structure representations: [evidence span withheld — anchor status: not_found]
- [readme] If no structure information is provided, it is queried from PubChem by Name search.: "If no structure information is provided, it is queried from PubChem by Name search."
- [readme] some need a local file and/or special access otherwise set it to False in the jobs.py: "For querying other databases, some need a local file and/or special access otherwise set it to False in the jobs.py:"
- [methods] Prefect flow deployment and work pool: "prefect deploy metadata_cleanup_prefect:cleanup_file --name local-deploy --pool local-work"
- [readme] metadata should follow a shared template with the same column names: "Please use the [template]...for your metadata for having same column names and minimum needed information for the query."
