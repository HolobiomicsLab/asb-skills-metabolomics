---
name: prefect-workflow-orchestration
description: Use when batch harmonizing multi-source compound metadata sheets — e.g.
  plate_id/well_location tables from natural-product and drug libraries — into a standardized
  template with structural descriptors.
when_to_use_negative:
- When the input already contains ChEMBL-standardized structures and all descriptors
  — the cleanup would redundantly re-standardize and re-query databases.
- When a quick one-off transformation is needed without job orchestration — serving
  a Prefect 2 server, work pool, and worker is unnecessary overhead.
- When structures are absent AND a name-to-structure PubChem Name lookup cannot resolve
  the compound — the flow cannot populate descriptors for unmatched entries.
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3314
tools:
- name: Prefect 2
  role: 'Workflow orchestrator: serves, deploys, and runs the cleanup_file flow with
    a local-work process pool (concurrency limit 5) that waits for jobs from jobs.py'
  repo: https://github.com/corinnabrungs/msn_tree_library
- name: prepare_wikidata_lotus_prefect.py
  role: Updates/prepares the LOTUS natural-product database file so the cleanup flow
    can query it
  repo: https://github.com/corinnabrungs/msn_tree_library
- name: drugbank_extraction.py
  role: Extracts DrugBank data from the downloaded release file so the cleanup flow
    can query it
  repo: https://github.com/corinnabrungs/msn_tree_library
- name: PubChem
  role: Fallback Name-search source that supplies structures for compounds missing
    structure information in the metadata
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41592-025-02813-0
    title: 'MSnLib: efficient generation of open multi-stage fragmentation mass spectral
      libraries'
  source_content_hash: sha256:ea64f9787a16ef9dbae99e2215a3db32a71c54c907acc35ff51504a88f22f69a
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
  - Prefect
  - Prefect 2
  - PubChem
  - drugbank_extraction.py
  - pip
  - prepare_wikidata_lotus_prefect.py
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3314
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# prefect-workflow-orchestration

## Summary

Serves the MSnLib metadata-cleanup pipeline as a Prefect 2 flow (cleanup_file) that accepts jobs from jobs.py, standardizing compound structures and computing descriptors (canonical/isomeric SMILES, InChI, InChIKey, logP, monoisotopic mass) across the seven compound libraries.

## When to use

Use when batch harmonizing multi-source compound metadata sheets — e.g. plate_id/well_location tables from natural-product and drug libraries — into a standardized template with structural descriptors.

## When NOT to use

- When the input already contains ChEMBL-standardized structures and all descriptors — the cleanup would redundantly re-standardize and re-query databases.
- When a quick one-off transformation is needed without job orchestration — serving a Prefect 2 server, work pool, and worker is unnecessary overhead.
- When structures are absent AND a name-to-structure PubChem Name lookup cannot resolve the compound — the flow cannot populate descriptors for unmatched entries.

## Inputs

- Metadata sheet in the shared Google-Sheets template (columns: unique sample id, plate_id, well_location, compound name, structure fields)
- jobs.py submissions (task definitions for the cleanup_file flow)
- Optional pre-fetched database files: LOTUS Wikidata file, Broad Drug Repurposing Hub, DrugCentral SQL dump
- Compound structures as SMILES or other representation strings for ChEMBL standardization

## Outputs

- Cleaned, standardized metadata table with one row per input compound
- Per-compound structural descriptors: canonical SMILES, isomeric SMILES, InChI, InChIKey, logP, monoisotopic mass
- Assigned unique sample ids with retained plate_id / well_location context

## How to apply

Deploy the cleanup flow locally: start the Prefect 2 server, create a 'local-work' process work pool, set its concurrency limit to 5, and deploy metadata_cleanup_prefect:cleanup_file with --name local-deploy. Start a worker on that pool, then serve the flow locally by running metadata_cleanup_prefect.py so it waits for jobs submitted from jobs.py. The flow harmonizes metadata to the shared template (unique sample id, plate_id, well_location), strips salts/counter-ions (a second pass is needed when the salt lacks a dot separator), standardizes structures through the ChEMBL structure pipeline, computes descriptors with RDKit/ChEMBL, and queries PubChem by Name for compounds missing structure. Databases without a local file or access (Dictionary of Natural Products, DrugBank) must be set to False in jobs.py; LOTUS data is refreshed beforehand with prepare_wikidata_lotus_prefect.py and DrugBank XML with drugbank_extraction.py.

## Related tools

- **Prefect 2** (Workflow orchestrator: serves, deploys, and runs the cleanup_file flow with a local-work process pool (concurrency limit 5) that waits for jobs from jobs.py) — https://github.com/corinnabrungs/msn_tree_library
- **prepare_wikidata_lotus_prefect.py** (Updates/prepares the LOTUS natural-product database file so the cleanup flow can query it) — https://github.com/corinnabrungs/msn_tree_library
- **drugbank_extraction.py** (Extracts DrugBank data from the downloaded release file so the cleanup flow can query it) — https://github.com/corinnabrungs/msn_tree_library
- **PubChem** (Fallback Name-search source that supplies structures for compounds missing structure information in the metadata)

## Examples

```
prefect deploy metadata_cleanup_prefect:cleanup_file --name local-deploy --pool local-work
```

## Evaluation signals

- Output table exists and contains one row per input compound (row count matches input metadata sheet).
- All standardized descriptor fields (canonical/isomeric SMILES, InChI, InChIKey, logP, monoisotopic mass) are populated for structured compounds.
- Every row carries a unique, non-null sample id.
- Salt/counter-ion removal completed without errors, including the required second pass for salts lacking a dot separator.
- Flow completes on the local-work pool without concurrency-limit violations (limit set to 5).

## Limitations

- Databases without a local file or special access (Dictionary of Natural Products, DrugBank, DrugCentral) must be set to False in jobs.py or queries fail.
- If no structure is provided, resolution depends entirely on PubChem Name search matching the compound name.
- Salt stripping requires a second pass when the salt component lacks a dot separator, otherwise the salt is not removed.

## Evidence

- [methods] metadata cleanup flow: "Start serving the flow locally by running `metadata_cleanup_prefect.py`, the service will then run
  and wait for jobs to be submitted (jobs.py)"
- [methods] deployment of prefect flow: "prefect deploy metadata_cleanup_prefect:cleanup_file --name local-deploy --pool local-work"
- [methods] worker pool creation and configuration: "prefect work-pool create --type process local-work
prefect work-pool update --concurrency-limit 5 local-work"
- [intro] PubChem fallback: "If no structure information is provided, it is queried from PubChem by Name search."
- [intro] database access requirements: "some need a local file and/or special access otherwise set it to False in the jobs.py"
