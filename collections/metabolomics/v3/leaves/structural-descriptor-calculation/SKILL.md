---
name: structural-descriptor-calculation
description: 'Use when a compound metadata sheet (template columns: unique sample
  id, plate_id, well_location, name, structure fields) must be standardized and enriched
  with SMILES/InChI/InChIKey/logP/monoisotopic mass before spectral library generation
  or database cross-referencing.'
when_to_use_negative:
- The input already contains standardized structure descriptors and only needs mass-spectrometry
  processing (use the mzmine high-throughput pipeline instead)
- The goal is fragment-spectrum matching or library search rather than metadata cleanup
  — this skill does not handle MSn spectra
- 'No structure or compound names are available at all: PubChem lookup relies on a
  Name search, so rows with neither structure nor name cannot be resolved'
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0154
- http://edamontology.org/topic_3314
tools:
- name: Prefect 2
  role: Orchestrates the metadata cleanup as a locally served flow (cleanup_file deployment,
    local-work pool, concurrency limit 5) that waits for jobs submitted from jobs.py
- name: prepare_wikidata_lotus_prefect.py
  role: Optional enrichment script that updates LOTUS data from Wikidata when a provided
    file is not used
  repo: https://github.com/corinnabrungs/msn_tree_library
- name: drugbank_extraction.py
  role: Optional script to extract drug information from the downloaded DrugBank release
  repo: https://github.com/corinnabrungs/msn_tree_library
- name: PubChem
  role: Name-search fallback to retrieve structure information for compounds whose
    metadata sheet lacks structure fields
- name: Dictionary of Natural Products
  role: Optional natural-product database queried for structures when local access
    is available
- name: LOTUS
  role: Optional natural-product database refreshed via prepare_wikidata_lotus_prefect.py
    or a provided file
- name: Broad Institute Drug Repurposing Hub
  role: Optional drug-information database downloaded from the Broad repo-hub
- name: DrugBank
  role: Optional drug database requiring access; the release is downloaded and processed
    with drugbank_extraction.py
- name: DrugCentral
  role: Optional drug database queried from its SQL dump file
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41592-025-02813-0
    title: 'MSnLib: efficient generation of open multi-stage fragmentation mass spectral
      libraries'
  source_content_hash: sha256:81a76f6136caa61f94123ae2a656ac672034f1c4c6ef4f5013d5364fc62b93bc
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
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_0154
  - http://edamontology.org/topic_3314
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7222222222222222
n_claims: 18
---

# structural-descriptor-calculation

## Summary

Compute per-compound structural descriptors — canonical and isomeric SMILES, InChI, InChIKey, logP, and monoisotopic mass — from a harmonized metadata table as part of the MSnLib metadata cleanup Prefect flow, using the ChEMBL structure pipeline for standardization and PubChem name lookups when structures are missing.

## When to use

Use when a compound metadata sheet (template columns: unique sample id, plate_id, well_location, name, structure fields) must be standardized and enriched with SMILES/InChI/InChIKey/logP/monoisotopic mass before spectral library generation or database cross-referencing.

## When NOT to use

- The input already contains standardized structure descriptors and only needs mass-spectrometry processing (use the mzmine high-throughput pipeline instead)
- The goal is fragment-spectrum matching or library search rather than metadata cleanup — this skill does not handle MSn spectra
- No structure or compound names are available at all: PubChem lookup relies on a Name search, so rows with neither structure nor name cannot be resolved

## Inputs

- Raw compound metadata sheet in the shared template format (unique sample id, plate_id, well_location, compound name, structure fields)
- Optional local database files: Dictionary of Natural Products export, prepared LOTUS Wikidata file, Broad Drug Repurposing Hub download, DrugBank release, DrugCentral SQL dump

## Outputs

- Cleaned metadata table with one row per input compound, populated with standardized structure descriptor fields (canonical SMILES, isomeric SMILES, InChI, InChIKey, logP, monoisotopic mass)
- Unique non-null sample ids per row with retained plate_id and well_location context

## How to apply

Load the raw metadata sheet with pandas and harmonize its columns to the shared template, removing salt/counter-ion components from structure representations (running a second salt-stripping pass when a salt lacks a dot separator). Standardize structures through the ChEMBL structure pipeline (salt stripping, neutralization, canonicalization), then compute per-compound descriptors with RDKit/ChEMBL utilities: canonical SMILES, isomeric SMILES, InChI, InChIKey, logP, and monoisotopic mass. For rows lacking structure information, query the structure from PubChem by Name search; optionally augment with Dictionary of Natural Products, LOTUS (via prepare_wikidata_lotus_prefect.py), Broad Drug Repurposing Hub, DrugBank (drugbank_extraction.py), or DrugCentral, setting any database without a local file or special access to False in jobs.py. Assign a unique sample id per row while retaining plate_id/well_location, and assemble the cleaned table for downstream sequence creation and well visualization. Run the flow as Prefect 2 (cleanup_file deployment on a local-work pool, concurrency limit 5) by serving metadata_cleanup_prefect.py locally and submitting jobs from jobs.py.

## Related tools

- **Prefect 2** (Orchestrates the metadata cleanup as a locally served flow (cleanup_file deployment, local-work pool, concurrency limit 5) that waits for jobs submitted from jobs.py)
- **prepare_wikidata_lotus_prefect.py** (Optional enrichment script that updates LOTUS data from Wikidata when a provided file is not used) — https://github.com/corinnabrungs/msn_tree_library
- **drugbank_extraction.py** (Optional script to extract drug information from the downloaded DrugBank release) — https://github.com/corinnabrungs/msn_tree_library
- **PubChem** (Name-search fallback to retrieve structure information for compounds whose metadata sheet lacks structure fields)
- **Dictionary of Natural Products** (Optional natural-product database queried for structures when local access is available)
- **LOTUS** (Optional natural-product database refreshed via prepare_wikidata_lotus_prefect.py or a provided file)
- **Broad Institute Drug Repurposing Hub** (Optional drug-information database downloaded from the Broad repo-hub)
- **DrugBank** (Optional drug database requiring access; the release is downloaded and processed with drugbank_extraction.py)
- **DrugCentral** (Optional drug database queried from its SQL dump file)

## Examples

```
prefect deploy metadata_cleanup_prefect:cleanup_file --name local-deploy --pool local-work
```

## Evaluation signals

- Output table exists and contains exactly one row per input compound (row count matches the input metadata sheet)
- All descriptor fields (SMILES, InChI, InChIKey, logP, monoisotopic mass) are populated per compound
- Sample ids are unique and non-null across the output table
- Field-presence checks pass on the standardized metadata template columns
- Prefect flow completes on the local-work pool with concurrency limit 5 without failed jobs

## Limitations

- Databases such as DrugBank require special access; without a local file or credentials they must be set to False in jobs.py
- Compounds with no structure and no resolvable name cannot be enriched, since the fallback relies on a PubChem Name search
- Salt stripping requires a second pass when salts lack a dot separator

## Evidence

- [methods] Standardization via ChEMBL pipeline with salt stripping and descriptor computation: [evidence span withheld — anchor status: not_found]
- [intro] PubChem fallback for missing structures: "If no structure information is provided, it is queried from PubChem by Name search."
- [intro] Database gating in jobs.py: "some need a local file and/or special access otherwise set it to False in the jobs.py"
- [methods] Prefect deployment commands: "prefect deploy metadata_cleanup_prefect:cleanup_file --name local-deploy --pool local-work"
- [methods] Prefect worker pool setup: "prefect work-pool create --type process local-work"
- [readme] Template metadata requirement: "Please use the [template]...for your metadata for having same column names and minimum needed information for the query."
- [methods] Prefect flow serving: "Start serving the flow locally by running `metadata_cleanup_prefect.py`, the service will then run
  and wait for jobs to be submitted (jobs.py)"
