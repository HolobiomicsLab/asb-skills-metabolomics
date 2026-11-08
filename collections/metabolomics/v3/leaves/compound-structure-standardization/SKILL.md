---
name: compound-structure-standardization
description: Use when a metadata sheet of small-molecule samples needs a shared template
  structure with same column names, standardized structures, and unique sample ids
  before spectral-library or acquisition work.
when_to_use_negative:
- The metadata sheet already follows the shared template with standardized structures
  and populated SMILES/InChI/InChIKey fields — re-running cleanup only duplicates
  work.
- Structures are already salt-stripped and canonicalized and no descriptor computation
  (logP, monoisotopic mass) is needed.
- The task is mass-spectral data acquisition or processing (e.g. Orbitrap ID-X sequence
  creation or mzmine processing) rather than metadata harmonization — this skill operates
  on the metadata sheet only.
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0605
- http://edamontology.org/topic_3314
tools:
- name: Prefect 2
  role: Orchestrates the cleanup_file flow; deployed locally and served via metadata_cleanup_prefect.py
    with a local-work work pool
  repo: corinnabrungs/msn_tree_library
- name: prepare_wikidata_lotus_prefect.py
  role: Prepares/updates the LOTUS natural-product database used for optional structure
    lookups
  repo: corinnabrungs/msn_tree_library
- name: drugbank_extraction.py
  role: Extracts DrugBank data for optional drug-information lookups
  repo: corinnabrungs/msn_tree_library
- name: PubChem
  role: 'Fallback source: queries the structure by Name search when no structure information
    is provided'
- name: pip
  role: Installs requirements for running the cleanup flow
  repo: corinnabrungs/msn_tree_library
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41592-025-02813-0
    title: 'MSnLib: efficient generation of open multi-stage fragmentation mass spectral
      libraries'
  source_content_hash: sha256:24b026ea694507af5b119a1e3009730852751cddbf3ae1cc7dc03e41850de5dc
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
  - http://edamontology.org/topic_0605
  - http://edamontology.org/topic_3314
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# compound-structure-standardization

## Summary

Harmonizes metadata sheets of compound libraries by running structures through the ChEMBL structure pipeline for salt stripping and canonicalization, then computes RDKit/ChEMBL descriptors (canonical/isomeric SMILES, InChI, InChIKey, logP, monoisotopic mass) and back-fills missing structures from PubChem name search.

## When to use

Use when a metadata sheet of small-molecule samples needs a shared template structure with same column names, standardized structures, and unique sample ids before spectral-library or acquisition work.

## When NOT to use

- The metadata sheet already follows the shared template with standardized structures and populated SMILES/InChI/InChIKey fields — re-running cleanup only duplicates work.
- Structures are already salt-stripped and canonicalized and no descriptor computation (logP, monoisotopic mass) is needed.
- The task is mass-spectral data acquisition or processing (e.g. Orbitrap ID-X sequence creation or mzmine processing) rather than metadata harmonization — this skill operates on the metadata sheet only.

## Inputs

- raw metadata sheet (template-format columns: unique sample id, plate_id, well_location, compound name, structure fields)
- compound structure representations containing salts/counter-ions
- optional local database files (LOTUS via prepare_wikidata_lotus_prefect.py, DrugBank via drugbank_extraction.py)

## Outputs

- cleaned/standardized metadata table with one row per input compound
- per-compound structural descriptors: canonical SMILES, isomeric SMILES, InChI, InChIKey, logP, monoisotopic mass
- unique non-null sample ids retained with plate_id / well_location context

## How to apply

Load the raw metadata sheet with pandas, harmonize it to the shared metadata template (same column names, minimum needed information) and remove salt/counter-ion components from structure representations, requiring a second pass when salts lack a dot. Run structures through the ChEMBL structure pipeline to standardize them (salt stripping, neutralization, canonicalization), then compute canonical SMILES, isomeric SMILES, InChI, InChIKey, logP and monoisotopic mass per compound. For entries lacking structure information, query the structure from PubChem by Name search; optionally enable ChEMBL, Dictionary of Natural Products, LOTUS, Broad Drug Repurposing Hub, DrugBank or DrugCentral lookups only when a local file or special access exists, otherwise set those databases to False in jobs.py. Assign a unique sample id per row and retain plate_id / well_location context; execute the whole flow as a Prefect 2 deployment (cleanup_file, local-work pool, concurrency limit 5).

## Related tools

- **Prefect 2** (Orchestrates the cleanup_file flow; deployed locally and served via metadata_cleanup_prefect.py with a local-work work pool) — corinnabrungs/msn_tree_library
- **prepare_wikidata_lotus_prefect.py** (Prepares/updates the LOTUS natural-product database used for optional structure lookups) — corinnabrungs/msn_tree_library
- **drugbank_extraction.py** (Extracts DrugBank data for optional drug-information lookups) — corinnabrungs/msn_tree_library
- **PubChem** (Fallback source: queries the structure by Name search when no structure information is provided)
- **pip** (Installs requirements for running the cleanup flow) — corinnabrungs/msn_tree_library

## Examples

```
prefect deploy metadata_cleanup_prefect:cleanup_file --name local-deploy --pool local-work
```

## Evaluation signals

- Output table exists and contains one row per input compound — row count matches the input metadata sheet.
- Standardized descriptor fields (SMILES, InChI, InChIKey, logP, monoisotopic mass) are populated for every row.
- Every sample id is unique and non-null, with plate_id / well_location retained.
- Structures are salt-stripped and canonicalized (no counter-ion components remain).
- Entries without structure information were successfully resolved via PubChem Name search or left explicitly flagged.

## Limitations

- Some external databases (Dictionary of Natural Products, DrugBank, DrugCentral) require a local file and/or special access; without it they must be set to False in jobs.py.
- Salt removal may require a second pass when salts lack a dot in the structure representation.
- Compounds without structure can only be back-filled from PubChem if a name is available.

## Evidence

- [other] standardizes structures with the ChEMBL structure pipeline and computes canonical/isomeric SMILES, InChI, InChIKey, logP and monoisotopic mass; compounds without structure are queried from PubChem by name: [evidence span withheld — anchor status: not_found]
- [intro] metadata template requirement: "Please use the [template]...for your metadata for having same column names and minimum needed information for the query."
- [intro] PubChem fallback: "If no structure information is provided, it is queried from PubChem by Name search."
- [intro] database access caveats: "some need a local file and/or special access otherwise set it to False in the jobs.py"
- [methods] Prefect deployment: "prefect deploy metadata_cleanup_prefect:cleanup_file --name local-deploy --pool local-work"
- [methods] work pool configuration: "prefect work-pool create --type process local-work"
