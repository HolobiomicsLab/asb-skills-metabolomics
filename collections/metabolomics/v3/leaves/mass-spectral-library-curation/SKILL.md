---
name: mass-spectral-library-curation
description: Use when building or updating an open spectral library from a plate-based
  high-throughput LC-MSn acquisition, where compound metadata must be reconciled with
  PubChem/LOTUS/DrugBank identifiers and spectra merged into InChIKey-indexed MSn
  trees.
when_to_use_negative:
- When the goal is standard single-stage MS/MS library building from legacy spectra
  without metadata reconciliation — MSnLib is designed for multi-stage MSn tree acquisition.
- When the source data are already vendor-converted open formats (mzML/MGF) and no
  metadata cleanup is needed; the Prefect cleanup flow adds no value.
- When none of the drug or natural product databases (LOTUS, DrugBank, Broad Hub,
  Dictionary of Natural Products) have been downloaded and no structure annotations
  are supplied — the flow will simply skip those sources.
edam_operation: http://edamontology.org/operation_0364
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: Prefect 2
  role: 'Orchestrates the metadata cleanup flow: server, deployment, work-pool creation,
    worker start, and job submission through jobs.py'
- name: prepare_wikidata_lotus_prefect.py
  role: Updates the LOTUS natural-product reference data used during metadata reconciliation
- name: drugbank_extraction.py
  role: Extracts drug records from the downloaded DrugBank SQL dump
- name: ThermoRawFileParser
  role: Converts Thermo Orbitrap ID-X RAW files to open formats (mzML/MGF) for mzmine
    processing
  repo: https://github.com/CompOmics/ThermoRawFileParser
- name: mzmine
  role: Hosts the high-throughput data acquisition and processing pipeline that builds
    the MSn trees
  repo: https://github.com/mzmine/mzmine
- name: Xcalibur / Orbitrap ID-X
  role: Generates the acquisition sequence from the cleaned metadata sheet (sample_id,
    plate_id, vial/well)
- name: MassBank-data
  role: Reference record repository validated for MSnLib's InChIKey-block overlap
    comparison
  repo: https://github.com/MassBank/MassBank-data
- name: pip / requirements
  role: Installs the Python dependencies required to run the cleanup flow
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41592-025-02813-0
    title: 'MSnLib: efficient generation of open multi-stage fragmentation mass spectral
      libraries'
  source_content_hash: sha256:84c4d167f349b92cb62de9b77f93ba934cc6e580bf45ce20098293e71d32e019
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/CompOmics/ThermoRawFileParser
  tools:
  - MassBank-data
  - MassBank/MassBank-data
  - Prefect 2
  - ThermoRawFileParser
  - Xcalibur / Orbitrap ID-X
  - corinnabrungs/msn_tree_library
  - drugbank_extraction.py
  - mzmine
  - pip
  - pip / requirements
  - prepare_wikidata_lotus_prefect.py
  edam_operation: http://edamontology.org/operation_0364
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5333333333333333
n_claims: 15
---

# mass-spectral-library-curation

## Summary

Build an open multi-stage (MSn) fragmentation spectral library by cleaning compound metadata against PubChem, LOTUS, and drug databases, then merging spectra from Orbitrap ID-X MS2/MS3 acquisitions into machine-ready MSn tree libraries.

## When to use

Use when building or updating an open spectral library from a plate-based high-throughput LC-MSn acquisition, where compound metadata must be reconciled with PubChem/LOTUS/DrugBank identifiers and spectra merged into InChIKey-indexed MSn trees.

## When NOT to use

- When the goal is standard single-stage MS/MS library building from legacy spectra without metadata reconciliation — MSnLib is designed for multi-stage MSn tree acquisition.
- When the source data are already vendor-converted open formats (mzML/MGF) and no metadata cleanup is needed; the Prefect cleanup flow adds no value.
- When none of the drug or natural product databases (LOTUS, DrugBank, Broad Hub, Dictionary of Natural Products) have been downloaded and no structure annotations are supplied — the flow will simply skip those sources.

## Inputs

- Metadata spreadsheet following the MSnLib template (sample_id, plate_id, vial/well location, structure information)
- PubChem responses from Name search when structure information is missing
- LOTUS Wikidata dump (or the pre-provided file)
- DrugBank latest release SQL dump
- Broad Institute Drug Repurposing Hub download
- Thermo Orbitrap ID-X RAW files acquired from the plate sequence

## Outputs

- Cleaned compound metadata with reconciled identifiers and structures
- Open multi-stage fragmentation spectral library (MSnLib): >2 million spectra in MSn trees of 30,008 unique small molecules
- First-InChIKey-block coverage counts relative to GNPS, MassBank EU, MoNA, mzCloud, and NIST23 (~22,700 compounds unique to MSnLib)
- Well-plate visualization (piecharts) for compound mixture wells

## How to apply

Start from a metadata spreadsheet using the shared template so that column names and the minimum query information match; if structure fields are absent, query PubChem by name search. Run the Prefect 2 metadata cleanup flow (`prefect server start`, deploy `cleanup_file`, create a `local-work` process work-pool with concurrency limit 5, start the worker) and submit entries through `jobs.py`, disabling databases that need local files or special access (Dictionary of Natural Products, DrugBank) by setting them to False. For LOTUS updates run `prepare_wikidata_lotus_prefect.py`; for DrugBank run `drugbank_extraction.py` on the downloaded SQL release; for the Broad Drug Repurposing Hub download the repo-hub dump directly. Assign each compound a unique sample_id with plate_id (batch identifier) and vial/well location, drive the Xcalibur/Orbitrap ID-X sequence from that sheet, and optionally render well-plate piecharts for compound mixtures. Convert Thermo RAW files with ThermoRawFileParser and process in mzmine to yield MSn tree spectra that merge with existing MassBank, GNPS, MoNA, mzCloud, and NIST23 records.

## Related tools

- **Prefect 2** (Orchestrates the metadata cleanup flow: server, deployment, work-pool creation, worker start, and job submission through jobs.py)
- **prepare_wikidata_lotus_prefect.py** (Updates the LOTUS natural-product reference data used during metadata reconciliation)
- **drugbank_extraction.py** (Extracts drug records from the downloaded DrugBank SQL dump)
- **ThermoRawFileParser** (Converts Thermo Orbitrap ID-X RAW files to open formats (mzML/MGF) for mzmine processing) — https://github.com/CompOmics/ThermoRawFileParser
- **mzmine** (Hosts the high-throughput data acquisition and processing pipeline that builds the MSn trees) — https://github.com/mzmine/mzmine
- **Xcalibur / Orbitrap ID-X** (Generates the acquisition sequence from the cleaned metadata sheet (sample_id, plate_id, vial/well))
- **MassBank-data** (Reference record repository validated for MSnLib's InChIKey-block overlap comparison) — https://github.com/MassBank/MassBank-data
- **pip / requirements** (Installs the Python dependencies required to run the cleanup flow)

## Examples

```
prefect deploy metadata_cleanup_prefect:cleanup_file --name local-deploy --pool local-work
```

## Evaluation signals

- The cleaned metadata sheet must retain the same column names as the shared template and contain the minimum needed information for the query (sample_id, plate_id, vial/well location, structure or PubChem-derived InChIKey).
- Prefect work-pool runs with concurrency limit 5 and processes all jobs submitted via jobs.py without failures; entries for unavailable databases are set to False rather than erroring.
- The resulting library contains MSn trees (not just flat MS/MS spectra) with unique first-InChIKey blocks; overlap counts against GNPS, MassBank EU, MoNA, mzCloud, and NIST23 should reproduce ~22,700 unique first-InChIKey blocks for MSnLib.
- Final library statistics should match the published figures: >2 million spectra across 30,008 unique small molecules.
- Well-plate piecharts correctly reflect compound mixtures when multiple compounds are present in a well.

## Limitations

- Some databases (Dictionary of Natural Products, DrugBank) require local file access or special credentials; if unavailable they must be set to False in jobs.py, which reduces the breadth of metadata reconciliation.
- The LOTUS preparation script must be run to obtain an up-to-date reference, otherwise only the pre-provided file is used.
- The sequence creation is specifically set up for the Orbitrap ID-X (Xcalibur); other instruments require their own acquisition setup.
- Metadata without structure information relies on PubChem Name search, which may return ambiguous or missing structures.

## Evidence

- [methods] Metadata cleanup flow: "Start serving the flow locally by running `metadata_cleanup_prefect.py`, the service will then run and wait for jobs to be submitted (jobs.py)"
- [methods] Deployment of prefect flow: "prefect deploy metadata_cleanup_prefect:cleanup_file --name local-deploy --pool local-work"
- [methods] Worker pool configuration: "prefect work-pool create --type process local-work; prefect work-pool update --concurrency-limit 5 local-work"
- [readme] Metadata template requirement: "Please use the template for your metadata for having same column names and minimum needed information for the query."
- [readme] PubChem fallback: "If no structure information is provided, it is queried from PubChem by Name search."
- [readme] LOTUS update script: "LOTUS: run `prepare_wikidata_lotus_prefect.py` for updating the data"
- [readme] DrugBank extraction script: "run `drugbank_extraction.py` on that file"
- [abstract] Library scope: [evidence span withheld — anchor status: not_found]
- [abstract] Pipeline tool: "built with a high-throughput data acquisition and processing pipeline in the open-source software mzmine"
- [methods] Sequence setup: "The sequence creation is setup for the Orbitrap ID-X (Xcalibur)."
