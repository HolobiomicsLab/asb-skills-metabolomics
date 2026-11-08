---
name: depth-first-block-extraction-from-molecular-graph
description: Use when input is a SMILES structure of an NRP/PK (e.g., from PubChem)
  that must be decomposed into residue-level building blocks with formulas and masses
  for CycloBranch spectra annotation.
when_to_use_negative:
- The input is already a sequence or building-block database (plain text for CycloBranch)
  rather than a SMILES structure to be decomposed
- The downstream tool cannot accept custom building blocks — e.g., NRPro users cannot
  create custom databases of building blocks and NRP sequences
- Only raw MS/MS spectra annotation is needed with an existing theoretical spectra
  tool (e.g., Dereplicator) and no custom block decomposition is required
edam_operation: http://edamontology.org/operation_0226
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_2258
tools:
- name: MassSpecBlocks
  role: Web tool that parses input SMILES into a molecular graph, annotates peptide/ester
    bonds, runs DFS to extract building blocks, and exports sequences/blocks for CycloBranch
  repo: github.com/privrja/MassSpecBlocks
- name: SmilesDrawer
  role: Modified library used to parse the SMILES into the molecular graph (vertices
    and edges arrays) that the DFS traverses
  repo: https://github.com/privrja/smilesDrawer
- name: Bbdgnc
  role: Previous application on which MassSpecBlocks' building-block generation and
    local-database lookup logic is based
  repo: https://github.com/privrja/bbdgnc
- name: PubChem
  role: Public database searched to retrieve the input structure in SMILES format
    (e.g., pseudacyclin A)
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1186/s13321-021-00530-2
    title: 'MassSpecBlocks: a web-based tool to create building blocks and sequences
      of nonribosomal peptides and polyketides for tandem mass spectra analysis'
  source_content_hash: sha256:b83942432796fa4443b9938e393edd90df30a82f662ab59e87cff0dc5bef80a8
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Bbdgnc
  - MassSpecBlocks
  - PubChem
  - SmilesDrawer
  edam_operation: http://edamontology.org/operation_0226
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_2258
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# depth-first-block-extraction-from-molecular-graph

## Summary

Decomposes a nonribosomal peptide or polyketide SMILES into ordered building blocks by parsing the SMILES into a molecular graph (vertices/edges via a modified SmilesDrawer), marking peptide/ester bonds, and running a depth-first search to emit block SMILES, formulas, and CycloBranch sequence notation.

## When to use

Use when input is a SMILES structure of an NRP/PK (e.g., from PubChem) that must be decomposed into residue-level building blocks with formulas and masses for CycloBranch spectra annotation.

## When NOT to use

- The input is already a sequence or building-block database (plain text for CycloBranch) rather than a SMILES structure to be decomposed
- The downstream tool cannot accept custom building blocks — e.g., NRPro users cannot create custom databases of building blocks and NRP sequences
- Only raw MS/MS spectra annotation is needed with an existing theoretical spectra tool (e.g., Dereplicator) and no custom block decomposition is required

## Inputs

- SMILES string of an NRP/PK chemical structure (retrievable by name, molecular formula, monoisotopic mass, database identifier, or SMILES from PubChem, ChemSpider, ChEBI, NP Atlas, COCONUT, or Norine)
- Molecular graph represented as an array of vertices (atoms) and an array of edges (bonds), as parsed by the modified SmilesDrawer library
- Local database container of building-block metadata (SMILES strings, acronyms, formulas) searched before assigning new block properties

## Outputs

- Building-block SMILES strings ordered from N-terminus to C-terminus
- Sequence in CycloBranch notation (e.g., [A]\[B]-[C]\[D]-[E] for a singly branched/branch-cyclic structure)
- Structure type classification (linear, cyclic, branched, branch-cyclic, or polyketide)
- Residue molecular formulas and monoisotopic masses per building block
- CSV export of blocks, sequence, and structure type (e.g., pseudacyclin_a_blocks.csv) in the plain-text format used by CycloBranch

## How to apply

Parse the SMILES into a molecular graph of vertices (atoms) and edges (bonds) using the modified SmilesDrawer parser, then traverse the edge array to annotate peptide and ester bonds, skipping any candidate bond whose resulting building block would contain three or fewer nonhydrogen vertices (so the Ac-Ile bond in pseudacyclin A stays unmarked). Run a depth-first search (DFS) over the annotated graph to generate building-block SMILES in N-terminus to C-terminus order, fixing carboxyl groups at C-termini instead of formyl groups. Classify topology from DFS cues: a block found more than once marks the structure cyclic, a block with more than two neighbors marks it branched, and a block lacking a C-terminus marks it a polyketide; emit the sequence in CycloBranch notation such as [A]\[B]-[C]\[D]-[E] for singly branched/branch-cyclic structures. Compute each residue's molecular formula and monoisotopic mass from the block formula: normally by water elimination, but for blocks terminated by amines on both sides and lacking oxygen (e.g., putrescine) eliminate two hydrogens instead (Put: C4H12N2 molecular, C4H10N2 residue). Write blocks, sequence, and structure type to a CSV file and check the local database first for existing block metadata before assigning new properties.

## Related tools

- **MassSpecBlocks** (Web tool that parses input SMILES into a molecular graph, annotates peptide/ester bonds, runs DFS to extract building blocks, and exports sequences/blocks for CycloBranch) — github.com/privrja/MassSpecBlocks
- **SmilesDrawer** (Modified library used to parse the SMILES into the molecular graph (vertices and edges arrays) that the DFS traverses) — https://github.com/privrja/smilesDrawer
- **Bbdgnc** (Previous application on which MassSpecBlocks' building-block generation and local-database lookup logic is based) — https://github.com/privrja/bbdgnc
- **PubChem** (Public database searched to retrieve the input structure in SMILES format (e.g., pseudacyclin A))

## Examples

```
symfony server:start  # then query the REST API documented at http://localhost:8000/rest/doc
```

## Evaluation signals

- On pseudacyclin A, the output must classify the structure as branch-cyclic and decompose it into six building blocks: the cyclo(Pro-Ile-Ile-Orn-Phe) ring plus the Ac-Ile branch
- The peptide bond connecting Ac to Ile must remain unmarked because the resulting Ac block would contain only three nonhydrogen atoms
- For a diamine, oxygen-free block like putrescine, the residue formula must be C4H10N2 (elimination of two hydrogens from C4H12N2), not the standard water-elimination residue
- Blocks with no C-terminus must be flagged as polyketides; blocks with more than two neighbors must trigger the branched classification
- The exported sequence must round-trip into CycloBranch's plain-text format without errors

## Limitations

- Peptide bonds whose resulting block would contain three or fewer nonhydrogen atoms (e.g., Ac) are not automatically marked, so short acyl branches are left attached to the neighboring block
- The default residue-formula convention (water elimination) fails for building blocks terminated by amine groups on both sides and lacking oxygen, requiring the two-hydrogen elimination rule
- Before every backend test run the database must be deleted, created, migrated, and purged; tests cannot be run more times in a row

## Evidence

- [methods] DFS over annotated molecular graph: "The depth-first search (DFS) was implemented to browse the molecular graph and generate the structures of building blocks in SMILES format."
- [results] Ac bond skip rule: "The peptide bond connecting Ac to Ile on the branch was not marked because the resulting block Ac would contain only three nonhydrogen atoms."
- [abstract] Tool purpose: "we introduce MassSpecBlocks - an open-source and web-based tool that converts the input chemical structures in SMILES format into sequences of building blocks"
- [methods] SMILES parsing basis: "rivrja/smilesDrawer) from **Daniel Probst** to draw chemical structures from SMILES format"
- [readme] Application scope: "MassSpecBlocks: Database of Sequences and Building Blocks of Microbial Metabolites for Mass Spectra Analysis is an open-source web application"
- [methods] Local database lookup before block metadata assignment: "The SMILES strings of building blocks are searched in a local database first (i.e., in a selected container) to obtain the remaining properties of the blocks."
