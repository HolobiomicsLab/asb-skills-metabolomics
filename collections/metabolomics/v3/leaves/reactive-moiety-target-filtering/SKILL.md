---
name: reactive-moiety-target-filtering
description: Use when annotating MALDI-MSI or LC-MS features acquired with a derivatizing
  matrix (e.g. FMP-10), where the candidate target list must be restricted to metabolites
  containing moieties the matrix reacts with (e.g. phenolic hydroxyls, primary/secondary
  amines).
when_to_use_negative:
- Input is already a filtered target list or feature table — the filtering step has
  already been applied
- Data were acquired with a conventional (non-derivatizing) matrix producing standard
  [M+H]+/[M-H]- ions — no reactive-moiety restriction is warranted
- Structures lack machine-readable SMILES/SDF records, since the filter requires RDKit-parseable
  structures
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: Met-ID
  role: Annotation software whose derivatizing-matrix filter this skill reproduces;
    reads SQLite databases of HMDB/LIPID MAPS structures and functional groups
  repo: https://github.com/pbjarterot/Met-ID
- name: RDKit
  role: Parses metabolite SMILES into sanitized molecular objects and performs SMARTS
    substructure matching to count derivatizable sites
- name: HMDB
  role: Openly available metabolite structure database providing the candidate SMILES
    records to filter
- name: LIPID MAPS
  role: Openly available lipid structure database providing additional candidate records
    to filter
- name: SQLite
  role: Storage format Met-ID uses for parsed HMDB and LIPID MAPS databases; queried
    for metabolites and functional-group definitions
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/acs.analchem.5c00633
    title: 'Met-ID: An Open-Source Software for Comprehensive Annotation of Multiple
      On-Tissue Chemical Modifications in MALDI-MSI'
  source_content_hash: sha256:e07c48f8f315372221bf3d3cc8141b134cb5081b7a7a128f12d234d4e23b3914
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/pbjarterot/Met-ID
  tools:
  - HMDB
  - LIPID MAPS
  - Met-ID
  - RDKit
  - SQLite
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# reactive-moiety-target-filtering

## Summary

Filters metabolite database candidates (e.g. HMDB, LIPID MAPS) down to those bearing functional groups reactive toward a specific derivatizing matrix, by substructure-matching SMARTS patterns against parsed SMILES structures with RDKit. This mirrors how Met-ID 'utilizes structural information for derivatizing matrices to generate a subset of targets that contain only functional groups specific to the derivatization agent'.

## When to use

Use when annotating MALDI-MSI or LC-MS features acquired with a derivatizing matrix (e.g. FMP-10), where the candidate target list must be restricted to metabolites containing moieties the matrix reacts with (e.g. phenolic hydroxyls, primary/secondary amines).

## When NOT to use

- Input is already a filtered target list or feature table — the filtering step has already been applied
- Data were acquired with a conventional (non-derivatizing) matrix producing standard [M+H]+/[M-H]- ions — no reactive-moiety restriction is warranted
- Structures lack machine-readable SMILES/SDF records, since the filter requires RDKit-parseable structures

## Inputs

- Metabolite structure records (SMILES/SDF) from HMDB or LIPID MAPS, or Met-ID's parsed SQLite databases (list_metabolites, list_functional_groups tables)
- SMARTS patterns for the derivatization agent's target functional groups (e.g. FMP-10-reactive phenolic OH and primary/secondary amines)

## Outputs

- Per-compound derivatization-site counts (e.g. derivatization_site_counts.csv with accession, compound name, site count)
- Filtered target subset containing only derivatization-compatible metabolites for downstream MS1 mass matching

## How to apply

Download the structural records (SMILES/SDF) of the target database used by Met-ID (HMDB and LIPID MAPS, which Met-ID parses into SQLite databases), then parse each structure into sanitized RDKit molecules, neutralizing salts and skipping unparseable entries. Define SMARTS patterns enumerating the functional groups reactive toward the chosen derivatizing matrix — for FMP-10 these are phenolic hydroxyls and primary/secondary amines — and run substructure matching to count derivatizable sites per compound. Compounds with zero matches (e.g. the cyanoacrylate enbucrilate) are non-derivatizable and excluded from the derivatized-target subset, while compounds can legitimately have multiple sites since Met-ID 'is able to identify multiple derivatization sites on the same molecule'. Assemble accession, compound name, and site count per compound and validate against known reference results (FMP-10 four-isomer example: enbucrilate = 0, p-octopamine = 2, norfenefrine = 2, dopamine = 3 sites).

## Related tools

- **Met-ID** (Annotation software whose derivatizing-matrix filter this skill reproduces; reads SQLite databases of HMDB/LIPID MAPS structures and functional groups) — https://github.com/pbjarterot/Met-ID
- **RDKit** (Parses metabolite SMILES into sanitized molecular objects and performs SMARTS substructure matching to count derivatizable sites)
- **HMDB** (Openly available metabolite structure database providing the candidate SMILES records to filter)
- **LIPID MAPS** (Openly available lipid structure database providing additional candidate records to filter)
- **SQLite** (Storage format Met-ID uses for parsed HMDB and LIPID MAPS databases; queried for metabolites and functional-group definitions)

## Examples

```
mols = [Chem.MolFromSmiles(s) for s in hmdb_smiles]; patt = Chem.MolFromSmarts('[OX2H]c;[NX3;H2,H3;!$(N*)]'); counts = [len(m.GetSubstructMatches(patt)) for m in mols if m is not None]
```

## Evaluation signals

- Four-isomer FMP-10 reference counts reproduced exactly: enbucrilate = 0 sites, p-octopamine = 2, norfenefrine = 2, dopamine = 3
- Every compound flagged non-derivatizable has zero SMARTS matches, and no derivatized-target assignment exists for it in downstream MS1 annotation
- Site counts are chemically plausible against drawn structures (e.g. dopamine: three reactive sites; two-site compounds have exactly two phenolic OH/amine groups)
- All SMILES entries parse to sanitized RDKit molecules or are explicitly skipped and logged; no silent failures
- Filtered subset is a strict subset of the original database and carries accession, name, and site count for every retained compound

## Limitations

- The FMP-10 derivatizing matrix was developed in-house; its availability/distribution to other users is not stated
- Met-ID's derivatization approach depends on rdkit, whose installation is nontrivial with no pinned environment or dependency file specified
- Unparseable or non-standard structure records must be skipped manually; neutralization/salt-stripping choices can alter site counts

## Evidence

- [abstract] derivatization filter uses structural information: "It utilizes structural information for derivatizing matrices to generate a subset of targets that contain only functional groups specific to the derivatization agent."
- [abstract] multiple derivatization sites supported: "The software is able to identify multiple derivatization sites on the same molecule, facilitating identification of the derivatized compound. This ability is exemplified by FMP-10"
- [discussion] SQLite databases of HMDB and LIPID MAPS: [evidence span withheld — anchor status: not_found]
- [methods] FMP-10 default in methods: "The defaults select ``HMDB(All)`` and the ``FMP-10`` matrix."
- [discussion] RDKit dependency and install difficulty: [evidence span withheld — anchor status: not_found]
