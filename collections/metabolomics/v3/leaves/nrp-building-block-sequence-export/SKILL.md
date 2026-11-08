---
name: nrp-building-block-sequence-export
description: Use when you need to create or share a custom NRP/PK sequence database
  (e.g. siderophores such as desferrioxamine B) for dereplication or comparator accuracy
  testing in CycloBranch, and no suitable ready-made database exists.
when_to_use_negative:
- If NRPro already satisfies your needs and you do not require a custom building block
  database — NRPro users cannot create custom databases of building blocks and NRP
  sequences, which is precisely why MassSpecBlocks was built
- If your structures are already annotated in a ready-made CycloBranch-compatible
  plain text database and no new NRP/PK sequences or building blocks need to be added
- If the input is a mass spectrum (mzML/mgf) rather than a chemical structure — this
  skill handles database construction, not spectra analysis
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0154
tools:
- name: MassSpecBlocks
  role: Web-based tool that converts SMILES chemical structures into sequences of
    building blocks and exports them into the CycloBranch plain text format
  repo: https://github.com/privrja/thesis-frontend-react
- name: CycloBranch
  role: Downstream tool that consumes the exported plain text sequence database to
    compare experimental tandem mass spectra against theoretical spectra
- name: smilesDrawer
  role: Embedded dependency (fork) used by MassSpecBlocks to draw chemical structures
    from SMILES format in the graphical interface
  repo: https://github.com/privrja/smilesDrawer
- name: bbdgnc
  role: Previous application on which MassSpecBlocks is based; demonstrates the building-block
    database generation lineage
  repo: https://github.com/privrja/bbdgnc
provenance:
  source_task_ids:
  - task_003
  - task_004
  source_papers:
  - doi: 10.1186/s13321-021-00530-2
    title: 'MassSpecBlocks: a web-based tool to create building blocks and sequences
      of nonribosomal peptides and polyketides for tandem mass spectra analysis'
  source_content_hash: sha256:4f37d287bd2c471400f026b342e2561891c0e5ddbefdc056ac2c7bf1e2090ed8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/privrja/thesis-frontend-react
  tools:
  - CycloBranch
  - MassSpecBlocks
  - bbdgnc
  - smilesDrawer
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7647058823529411
n_claims: 17
---

# nrp-building-block-sequence-export

## Summary

Constructs custom databases of nonribosomal peptide (NRP) and polyketide (PK) building blocks and sequences from SMILES chemical structures using the MassSpecBlocks web tool, then exports them into the plain text sequence database format consumed by CycloBranch for tandem mass spectra annotation.

## When to use

Use when you need to create or share a custom NRP/PK sequence database (e.g. siderophores such as desferrioxamine B) for dereplication or comparator accuracy testing in CycloBranch, and no suitable ready-made database exists.

## When NOT to use

- If NRPro already satisfies your needs and you do not require a custom building block database — NRPro users cannot create custom databases of building blocks and NRP sequences, which is precisely why MassSpecBlocks was built
- If your structures are already annotated in a ready-made CycloBranch-compatible plain text database and no new NRP/PK sequences or building blocks need to be added
- If the input is a mass spectrum (mzML/mgf) rather than a chemical structure — this skill handles database construction, not spectra analysis

## Inputs

- Chemical structures in SMILES format (from PubChem, ChemSpider, ChEBI, NP Atlas, COCONUT, Norine, or manual entry)
- Molecular formula, monoisotopic mass, database identifier, or compound name for searching
- Container of building blocks and sequences (e.g. MassSpecBlocks 'Nonribosomal Peptides and Siderophores' container)

## Outputs

- Sequences and building blocks in the plain text format used by CycloBranch
- Custom NRP/PK sequence and building block database stored in a MassSpecBlocks container
- Exported sequence database suitable for loading into CycloBranch comparator/finder searches

## How to apply

First search for the target molecular structure (SMILES format) across PubChem, ChemSpider, ChEBI, NP Atlas, COCONUT, or Norine from within MassSpecBlocks, or enter it manually. Trigger the 'Build Blocks' button, which performs a depth-first search (DFS) over the molecular graph to decompose the structure into building blocks in SMILES format; canonicalize the notation with the Unique SMILES button and strip markups with Generic SMILES when needed. Verify the resulting branch-cyclic or branched topology (e.g. pseudacyclin A should decompose into a cyclo(Pro-Ile-Ile-Orn-Phe) ring plus an Ac-Ile branch), then save the sequence and blocks into a container such as 'Nonribosomal Peptides and Siderophores' (85 building blocks, 146 sequences). Finally export the container's sequences and building blocks into the plain text format accepted by CycloBranch so the database can be loaded for comparing experimental tandem mass spectra against theoretical spectra.

## Related tools

- **MassSpecBlocks** (Web-based tool that converts SMILES chemical structures into sequences of building blocks and exports them into the CycloBranch plain text format) — https://github.com/privrja/thesis-frontend-react
- **CycloBranch** (Downstream tool that consumes the exported plain text sequence database to compare experimental tandem mass spectra against theoretical spectra)
- **smilesDrawer** (Embedded dependency (fork) used by MassSpecBlocks to draw chemical structures from SMILES format in the graphical interface) — https://github.com/privrja/smilesDrawer
- **bbdgnc** (Previous application on which MassSpecBlocks is based; demonstrates the building-block database generation lineage) — https://github.com/privrja/bbdgnc

## Evaluation signals

- The exported plain text file loads into CycloBranch without errors and all sequences/building blocks appear in the target database list
- Building block decomposition is chemically correct — e.g. for pseudacyclin A, MassSpecBlocks should report the branch-cyclic structure with a cyclo(Pro-Ile-Ile-Orn-Phe) ring and an Ac-Ile branch
- SMILES notation of each building block is canonicalized (Unique SMILES applied) and free of generic markups (Generic SMILES applied)
- When the exported database is used in a CycloBranch search with precursor mass filter enabled, the correct compound (e.g. desferrioxamine B) is reported as the single hit
- The exported database matches the expected container scale — 85 building blocks and 146 sequences for the default 'Nonribosomal Peptides and Siderophores' container

## Limitations

- The peptide bond connecting Ac to Ile on the branch was not automatically marked because the resulting Ac block would contain only three nonhydrogen atoms — manual inspection of the decomposition is required
- It is not always easy to export flawless sequences of building blocks for tandem mass spectra analysis from external databases such as Norine
- The building-block generation searches a local database first; if a SMILES string is not present locally, remaining properties of the block may need to be supplied manually
- Users cannot create custom databases of building blocks and NRP sequences in NRPro, so MassSpecBlocks is required when custom building blocks are needed

## Evidence

- [abstract] MassSpecBlocks converts SMILES into building-block sequences and exports them for CycloBranch: "converts the input chemical structures in SMILES format into sequences of building blocks"
- [abstract] Sequences and blocks export into CycloBranch plain text format: "The sequences and building blocks created in MassSpecBlocks can be easily exported into a plain text format used by CycloBranch"
- [methods] Build Blocks performs DFS over molecular graph to decompose structure: "A list of building blocks is generated using the button Build Blocks (Fig. 5). The SMILES strings of building blocks are searched in a local database first"
- [readme] Default container has 85 building blocks and 146 sequences: "MassSpecBlocks: Database of Sequences and Building Blocks of Microbial Metabolites for Mass Spectra Analysis is an open-source web application to manage own user databases of chemical structures like"
- [results] Task card: export 44 evaluation compounds into CycloBranch plain text format: [evidence span withheld — anchor status: not_found]
- [results] Pseudacyclin A decomposed correctly into branch-cyclic structure: "MassSpecBlocks correctly determined the branch-cyclic structure of the molecule."
- [intro] NRPro cannot create custom building block databases: "However, users cannot create custom databases of building blocks and NRP sequences."
