---
name: multi-database-metadata-enrichment
description: Use when a metadata sheet (unique sample id, plate_id, well_location,
  compound name, structure fields) for compound libraries lacks standardized structure
  information or descriptor fields and needs enrichment from PubChem plus optional
  DNP/LOTUS/DrugBank/DrugCentral/Broad-Hub lookups.
when_to_use_negative:
- When the metadata sheet already contains validated, standardized structures and
  all descriptor fields (SMILES/InChI/InChIKey/logP/monoisotopic mass) populated —
  re-running the cleanup adds no information
- When the goal is MS data acquisition or spectral library building (e.g. Orbitrap
  ID-X sequence creation or mzmine processing) rather than compound metadata/structure
  harmonization
- When a database requires a local file and/or special access that is unavailable
  and you cannot set it to False in jobs.py — the flow will not correctly skip that
  source
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_3314
- http://edamontology.org/topic_0091
tools:
- name: Prefect 2
  role: Orchestrates the metadata cleanup flow (server start, flow deployment, work-pool
    creation and concurrency-limit 5 worker) and serves it locally for job submission
- name: pip
  role: Installs the Python requirements for running the cleanup flow
- name: prepare_wikidata_lotus_prefect.py
  role: Updates the LOTUS natural-product data used for compound lookup (otherwise
    a provided file is used)
- name: drugbank_extraction.py
  role: Extracts drug information from the DrugBank release download for use as a
    query database
- name: PubChem
  role: Fallback structure source queried by Name search when no structure information
    is provided in the metadata sheet
- name: Dictionary of Natural Products
  role: Natural-product compound lookup source requiring access; set to False in jobs.py
    if unavailable
- name: LOTUS
  role: Natural-product lookup database prepared via prepare_wikidata_lotus_prefect.py
    or a provided file
- name: Broad Institute Drug Repurposing Hub
  role: Drug-information lookup source downloaded from the Broad repo-hub
- name: DrugBank
  role: Drug-information lookup source requiring access; its release file is processed
    with drugbank_extraction.py
- name: DrugCentral
  role: Drug-information lookup source used via its SQL dump file
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41592-025-02813-0
    title: 'MSnLib: efficient generation of open multi-stage fragmentation mass spectral
      libraries'
  source_content_hash: sha256:05c503ce8c480a56c3ca308bf51af9e465fc38d39a92f8b961c8bc06372f89af
schema_version: 0.2.0
metadata:
  license_tier: open
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
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_3314
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 12
---

# multi-database-metadata-enrichment

## Summary

Harmonizes a shared-template metadata sheet for compound libraries and enriches each row with standardized structures and descriptors (canonical/isomeric SMILES, InChI, InChIKey, logP, monoisotopic mass) by querying PubChem and optional natural-product/drug databases, orchestrated as a Prefect 2 flow with per-database toggles in jobs.py.

## When to use

Use when a metadata sheet (unique sample id, plate_id, well_location, compound name, structure fields) for compound libraries lacks standardized structure information or descriptor fields and needs enrichment from PubChem plus optional DNP/LOTUS/DrugBank/DrugCentral/Broad-Hub lookups.

## When NOT to use

- When the metadata sheet already contains validated, standardized structures and all descriptor fields (SMILES/InChI/InChIKey/logP/monoisotopic mass) populated — re-running the cleanup adds no information
- When the goal is MS data acquisition or spectral library building (e.g. Orbitrap ID-X sequence creation or mzmine processing) rather than compound metadata/structure harmonization
- When a database requires a local file and/or special access that is unavailable and you cannot set it to False in jobs.py — the flow will not correctly skip that source

## Inputs

- Metadata spreadsheet (template-format) with unique sample id, plate_id, well_location, compound name and structure fields
- LOTUS prepared Wikidata file (or output of prepare_wikidata_lotus_prefect.py)
- DrugBank latest-release download file (post drugbank_extraction.py)
- Broad Institute Drug Repurposing Hub download file
- DrugCentral SQL dump file
- Dictionary of Natural Products access file

## Outputs

- Cleaned/standardized metadata table with one row per input compound
- Per-compound structural descriptors: canonical SMILES, isomeric SMILES, InChI, InChIKey, logP, monoisotopic mass
- PubChem-derived structure for entries lacking structure information
- Unique, non-null sample ids with retained plate_id and well_location context

## How to apply

Load the raw metadata sheet with pandas and harmonize it to the shared template (same column names, minimum needed information: unique sample id, plate_id, well_location, compound name, structure fields). Remove salt/counter-ion components from structure representations, standardize structures with the ChEMBL structure pipeline (salt stripping, neutralization, canonicalization), then compute descriptors per compound with RDKit/ChEMBL utilities: canonical SMILES, isomeric SMILES, InChI, InChIKey, logP and monoisotopic mass. For rows without a structure, query PubChem by Name search; for additional natural-product and drug databases, run prepare_wikidata_lotus_prefect.py for LOTUS, drugbank_extraction.py on the DrugBank release file, and set databases without a local file or special access (e.g. Dictionary of Natural Products) to False in jobs.py. Execute the cleanup as a Prefect 2 flow: serve metadata_cleanup_prefect.py locally, deploy as 'prefect deploy metadata_cleanup_prefect:cleanup_file --name local-deploy --pool local-work', create a process work-pool 'local-work', update its concurrency limit to 5, start the worker, and submit jobs from jobs.py. Assign a unique sample id per row and retain plate_id / well_location context in the cleaned table.

## Related tools

- **Prefect 2** (Orchestrates the metadata cleanup flow (server start, flow deployment, work-pool creation and concurrency-limit 5 worker) and serves it locally for job submission)
- **pip** (Installs the Python requirements for running the cleanup flow)
- **prepare_wikidata_lotus_prefect.py** (Updates the LOTUS natural-product data used for compound lookup (otherwise a provided file is used))
- **drugbank_extraction.py** (Extracts drug information from the DrugBank release download for use as a query database)
- **PubChem** (Fallback structure source queried by Name search when no structure information is provided in the metadata sheet)
- **Dictionary of Natural Products** (Natural-product compound lookup source requiring access; set to False in jobs.py if unavailable)
- **LOTUS** (Natural-product lookup database prepared via prepare_wikidata_lotus_prefect.py or a provided file)
- **Broad Institute Drug Repurposing Hub** (Drug-information lookup source downloaded from the Broad repo-hub)
- **DrugBank** (Drug-information lookup source requiring access; its release file is processed with drugbank_extraction.py)
- **DrugCentral** (Drug-information lookup source used via its SQL dump file)

## Examples

```
prefect server start; prefect deploy metadata_cleanup_prefect:cleanup_file --name local-deploy --pool local-work; prefect work-pool create --type process local-work; prefect work-pool update --concurrency-limit 5 local-work; prefect worker start --pool local-work
```

## Evaluation signals

- Output table exists and contains one row per input compound (row count matches the input metadata sheet)
- Standardized structure descriptor fields (canonical SMILES, isomeric SMILES, InChI, InChIKey, logP, monoisotopic mass) are populated for every row
- Unique, non-null sample ids present for all rows with plate_id and well_location retained
- Salts/counter-ion components removed from structure representations after the ChEMBL structure pipeline pass
- Databases without a local file or special access are set to False in jobs.py and are not queried

## Limitations

- Some databases (Dictionary of Natural Products, DrugBank) need a local file and/or special access; without it they must be set to False in jobs.py and will not contribute lookups
- LOTUS data can go stale unless prepare_wikidata_lotus_prefect.py is re-run to update the Wikidata-derived file
- Compounds without any structure information depend entirely on a successful PubChem Name search; a failed name match yields no structure descriptors
- The cleanup runs as a Prefect 2 flow with a work-pool concurrency limit of 5, which constrains parallel job throughput

## Evidence

- [intro] Metadata should follow a shared template with the same column names and minimum needed information for the query: "Please use the [template]...for your metadata for having same column names and minimum needed information for the query."
- [intro] If no structure information is provided, it is queried from PubChem by Name search: "If no structure information is provided, it is queried from PubChem by Name search."
- [intro] Some external databases need a local file and/or special access, otherwise set them to False in jobs.py: "some need a local file and/or special access otherwise set it to False in the jobs.py"
- [methods] The cleanup flow is served locally and waits for jobs submitted from jobs.py: "Start serving the flow locally by running `metadata_cleanup_prefect.py`, the service will then run
  and wait for jobs to be submitted (jobs.py)"
- [methods] Deployment command for the Prefect cleanup flow: "```bash
prefect deploy metadata_cleanup_prefect:cleanup_file --name local-deploy --pool local-work
```"
- [methods] Work-pool creation, concurrency limit and worker start commands: "```bash
prefect work-pool create --type process local-work
prefect work-pool update --concurrency-limit 5 local-work
```"
- [intro] LOTUS update procedure: "LOTUS: run `prepare_wikidata_lotus_prefect.py` for updating the data (otherwise use the provided file)"
- [intro] DrugBank extraction procedure: "DrugBank (access needed): [Download](https://go.drugbank.com/releases/latest) and
run `drugbank_extraction.py` on that"
- [methods] Requirements for running the flow: "## Requirements for running:

```bash
pip install requirements
```"
