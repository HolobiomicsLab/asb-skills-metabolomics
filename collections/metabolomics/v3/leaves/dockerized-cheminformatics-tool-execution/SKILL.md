---
name: dockerized-cheminformatics-tool-execution
description: Use when a workflow needs a cheminformatics predictor or parser (e.g.,
  CFM-ID fragment prediction) applied batch-wise to many structures (SMILES/InChI/.mol)
  and a Docker image of the tool exists.
when_to_use_negative:
- You already have experimental MS/MS spectra — this skill only generates predicted
  fragments; experimental spectra of the 15 reference standards were acquired separately
- The tool you need has no maintained Docker image and you cannot pin its version
  — ad-hoc local installs undermine the reproducibility rationale of this skill
- The input structures are not in a computer-readable format (SMILES/InChI/.mol) —
  e.g., only IUPAC nomenclature is available without prior structure conversion
edam_operation: http://edamontology.org/operation_0324
tools:
- name: CFM-ID
  role: The dockerized predictor executed on each adduct structure to generate in
    silico MS/MS fragments at three energy levels
- name: R (DT package)
  role: 'Downstream consumer of the container output: assembles the >5% fragments
    into the searchable datatable with parent-adduct annotation'
  repo: https://gitlab.com/nexs-metabolomics/projects/dna_adductomics_database
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.3389/fchem.2022.908572
    title: A Comprehensive Database for DNA Adductomics
  source_content_hash: sha256:77fcd5508fe02c98f2465a560cfaaf928225c78ad4d35f19ffa4926f145bf0f2
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CFM-ID
  - DT
  - R
  - R (DT package)
  edam_operation: http://edamontology.org/operation_0324
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# dockerized-cheminformatics-tool-execution

## Summary

Run containerized cheminformatics tools (here the wishartlab/cfmid Docker image of CFM-ID) over a batch of chemical structures to produce computable outputs — in this case in silico MS/MS fragments for 582 DNA adducts at three energy levels — without local tool installation.

## When to use

Use when a workflow needs a cheminformatics predictor or parser (e.g., CFM-ID fragment prediction) applied batch-wise to many structures (SMILES/InChI/.mol) and a Docker image of the tool exists.

## When NOT to use

- You already have experimental MS/MS spectra — this skill only generates predicted fragments; experimental spectra of the 15 reference standards were acquired separately
- The tool you need has no maintained Docker image and you cannot pin its version — ad-hoc local installs undermine the reproducibility rationale of this skill
- The input structures are not in a computer-readable format (SMILES/InChI/.mol) — e.g., only IUPAC nomenclature is available without prior structure conversion

## Inputs

- Chemical structure files per entry: .mol files (plus .cdxml) for 582 DNA adducts
- Computer-readable structure identifiers from an .xlsx table: SMILES, InChI, InChIKey
- Docker image of the target tool (wishartlab/cfmid for CFM-ID)

## Outputs

- In silico MS/MS fragment predictions for all 582 adducts at three energy levels
- Per-fragment output records carrying relative intensity, later filtered to >5% and annotated with parent adduct

## How to apply

Obtain the deposited structure archive (here the .xlsx, .cdxml, and .mol files for the 582 DNA adduct entries from the dna_adductomics_database GitLab repo), then load the .xlsx and extract computer-readable identifiers (SMILES, InChI, InChIKey) plus each adduct's .mol file. Launch the vendor's Docker image — wishartlab/cfmid for CFM-ID — and run it on every structure, keeping parameters matched to the reference protocol (three energy levels for MS/MS prediction, mirroring the three-level scheme used for the experimental spectra). Collect the raw per-structure output, then apply downstream filters (retain fragment ions with relative intensity above the 5% threshold, one entry per fragment ion) in a separate step rather than inside the container. The rationale for containerization is reproducibility: pinned tool versions make the predicted-fragment datatable regenerable, and the 5% intensity cutoff keeps the searchable datatable compact. Validate by checking that outputs exist for all 582 adducts at three energy levels before assembling the datatable with R (DT/Tidyverse).

## Related tools

- **CFM-ID** (The dockerized predictor executed on each adduct structure to generate in silico MS/MS fragments at three energy levels)
- **R (DT package)** (Downstream consumer of the container output: assembles the >5% fragments into the searchable datatable with parent-adduct annotation) — https://gitlab.com/nexs-metabolomics/projects/dna_adductomics_database

## Examples

```
docker pull wishartlab/cfmid
```

## Evaluation signals

- The run produces fragment entries for all 582 adducts, not a subset
- Predictions cover exactly three energy levels, matching the three-level scheme of the experimental MS/MS spectra
- Every retained fragment ion has relative intensity above the 5% threshold and appears only once (one entry per fragment ion)
- Each fragment row is annotated with its parent adduct
- The pipeline is reproducible from the pinned wishartlab/cfmid Docker image and the deposited .xlsx/.mol archive

## Limitations

- CFM-ID predictions are in silico approximations; the paper pairs them with experimental MS/MS spectra of only 15 reference standards, so predicted fragments for the remaining adducts are unvalidated
- The rstudio/DT README notes the package is in maintenance-only mode and pull requests may not be merged — a downstream dependency risk
- Fragment output quality depends on the correctness of the input SMILES/InChI/.mol structures curated in the .xlsx archive

## Evidence

- [methods] Run the wishartlab/cfmid docker image of CFM-ID on each structure: [evidence span withheld — anchor status: not_found]
- [methods] filtering fragment ions above 5% relative intensity: [evidence span withheld — anchor status: not_found]
- [abstract] abstract statement of database with 582 entries: "The database, including 582 entries, is publicly available ( https://gitlab.com/nexs-metabolomics/projects/dna_adductomics_database )."
- [readme] README statement that DT is in maintenance-only mode: [evidence span withheld — anchor status: external_doc]
