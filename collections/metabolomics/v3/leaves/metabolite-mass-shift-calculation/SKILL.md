---
name: metabolite-mass-shift-calculation
description: Use when you need theoretical derivative m/z values for metabolites reacted
  with a derivatizing matrix (AMPP, FMP-10, or a custom reagent) and the reagent's
  mass shift per reactive site, e.g., for dopamine, GABA, spermidine panels.
when_to_use_negative:
- The input is already measured MALDI-MSI m/z features needing annotation against
  a database — use Met-ID's identify() workflow with recalibration and MS2 comparison
  instead of computing theoretical shifts
- The metabolites are underivatized and expected to ionize as standard [M+H]+ / [M-H]-
  adducts with no derivatizing matrix involved
- The reagent's reactive functional group or per-site mass shift is unknown and cannot
  be defined — substructure matching and shift addition cannot proceed
tools:
- name: Met-ID
  role: Provides the local server (MetIDServer), searchable metabolite database with
    SMILES (search_by_name), and the derivatization-site/mass-shift model (possible_derivs)
    that this skill extends to new reagents like AMPP
  repo: https://github.com/pbjarterot/Met-ID
- name: RDKit
  role: Substructure matching on SMILES to detect derivatization-reactive amine functional
    groups and count sites per molecule
  repo: https://www.rdkit.org/
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1021/acs.analchem.5c00633
    title: 'Met-ID: An Open-Source Software for Comprehensive Annotation of Multiple
      On-Tissue Chemical Modifications in MALDI-MSI'
  source_content_hash: sha256:8b1b0580d14ad9a4230dabdfc11bf427f86df7cdf34fdd5c4fbbb757b53bdeb3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/pbjarterot/Met-ID
  tools:
  - Met-ID
  - RDKit
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# metabolite-mass-shift-calculation

## Summary

Computes theoretical derivative m/z values by enumerating derivatization-reactive functional groups (e.g., amines) on a metabolite's SMILES and applying one reagent mass shift (e.g., AMPP, FMP-10) per reactive site, extending Met-ID beyond its preconfigured FMP-10 setup.

## When to use

Use when you need theoretical derivative m/z values for metabolites reacted with a derivatizing matrix (AMPP, FMP-10, or a custom reagent) and the reagent's mass shift per reactive site, e.g., for dopamine, GABA, spermidine panels.

## When NOT to use

- The input is already measured MALDI-MSI m/z features needing annotation against a database — use Met-ID's identify() workflow with recalibration and MS2 comparison instead of computing theoretical shifts
- The metabolites are underivatized and expected to ionize as standard [M+H]+ / [M-H]- adducts with no derivatizing matrix involved
- The reagent's reactive functional group or per-site mass shift is unknown and cannot be defined — substructure matching and shift addition cannot proceed

## Inputs

- Metabolite names or database records with SMILES strings (e.g., dopamine, GABA, spermidine from Met-ID's HMDB/LIPID MAPS SQLite backend)
- Derivatization reagent definition: reactive functional group (e.g., amines for AMPP) and per-site mass shift

## Outputs

- CSV table (e.g., ampp_derivatization_sites.csv) with metabolite name, SMILES, derivatization-site count (possible_derivs analog), and derivative m/z values
- Per-metabolite list of reactive substructure matches (RDKit substructure matching results)

## How to apply

Start a local Met-ID server using the MetIDServer context manager, then retrieve database records including SMILES for each target metabolite with search_by_name(name, ccs=False). Detect derivatization-reactive functional groups (for AMPP, primary/secondary amines) in each SMILES by RDKit substructure matching, and count the sites per molecule, analogous to Met-ID's possible_derivs field. Compute each derivative m/z by adding one reagent mass shift per reactive site to the metabolite's protonated mass [M+H]+. Assemble a table of metabolite name, SMILES, derivatization-site count, and derivative m/z values (e.g., ampp_derivatization_sites.csv) and validate by reading it back with pandas: every metabolite should be present with at least one annotated site and finite, mass-shift-consistent m/z. The rationale is that Met-ID itself uses structural information about the derivatizing matrix to subset targets by reactive functional groups, and the README sanctions extending this to any derivatizing matrix via local changes.

## Related tools

- **Met-ID** (Provides the local server (MetIDServer), searchable metabolite database with SMILES (search_by_name), and the derivatization-site/mass-shift model (possible_derivs) that this skill extends to new reagents like AMPP) — https://github.com/pbjarterot/Met-ID
- **RDKit** (Substructure matching on SMILES to detect derivatization-reactive amine functional groups and count sites per molecule) — https://www.rdkit.org/

## Examples

```
with MetIDServer() as metid: rec = metid.search_by_name('dopamine', ccs=False)
```

## Evaluation signals

- All panel metabolites (dopamine, GABA, spermidine) are present in the output table after pandas readback
- Each metabolite has at least one annotated derivatization site (site count ≥ 1, matching its number of reactive amine groups)
- All derivative m/z values are finite and equal the protonated metabolite mass plus n × (AMPP per-site mass shift), where n = derivatization-site count
- Site counts are reproducible across RDKit substructure-matching runs on the same SMILES
- The theoretical derivative m/z values fall within the range of Met-ID database entries for derivatized compounds when cross-checked against list_metabolites/list_functional_groups

## Limitations

- FMP-10 was developed in-house and is preconfigured; other reagents like AMPP require user-side extension, and all database changes affect only the local Met-ID install
- Reinstalling Met-ID does not update bundled database files; the 'com.farmbio.metid' app-data folder must be removed manually for the intended database files
- RDKit installation is described as slightly more difficult than an average PyPI project (often works with anaconda), and no pinned dependency versions are stated

## Evidence

- [readme] Met-ID is extendable to use any derivatizing matrix with the tools to do local version changes right from inside the software.: [evidence span withheld — anchor status: external_doc]
- [methods] Import ``MetIDServer`` and use it as a context manager.: "Import ``MetIDServer`` and use it as a context manager."
- [abstract] The software is able to identify multiple derivatization sites on the same molecule, facilitating identification of the derivatized compound.: "The software is able to identify multiple derivatization sites on the same molecule, facilitating identification of the derivatized compound."
- [intro] It utilizes structural information for derivatizing matrices to generate a subset of targets that contain only functional groups specific to the derivatization agent.: "It utilizes structural information for derivatizing matrices to generate a subset of targets that contain only functional groups specific to the derivatization agent."
