---
name: nrp-sequence-topology-classification
description: Use when a natural-product SMILES must be decomposed into ordered building
  blocks and its topology (cyclic vs branched vs branch-cyclic vs polyketide) must
  be determined for CycloBranch sequence databases.
when_to_use_negative:
- The goal is de novo interpretation of a tandem mass spectrum without a known structure
  — use CycloBranch/dereplication tools instead of structure-driven decomposition
- The input is a pre-existing building-block/sequence database rather than a SMILES
  structure to decompose
- Custom building-block metadata must come from a local container lookup first (Build
  Blocks searches the local database before regenerating properties)
edam_topics:
- http://edamontology.org/topic_0154
tools:
- name: MassSpecBlocks
  role: Web-based tool that converts input SMILES structures into sequences of building
    blocks via bond annotation and depth-first search, and classifies topology
  repo: https://github.com/privrja/MassSpecBlocks
- name: SmilesDrawer (modified)
  role: Parses SMILES into the molecular graph (vertices and edges arrays) used for
    the DFS decomposition
  repo: https://github.com/privrja/smilesDrawer
- name: CycloBranch
  role: Consumes the exported plain-text sequence/building-block notation for annotating
    spectra of linear, cyclic, branched, and branch-cyclic NRPs
  repo: https://ms.biomed.cas.cz/cyclobranch/docs/html/
- name: PubChem
  role: Public database to retrieve the input structure by name, molecular formula,
    monoisotopic mass, identifier, or SMILES
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1186/s13321-021-00530-2
    title: 'MassSpecBlocks: a web-based tool to create building blocks and sequences
      of nonribosomal peptides and polyketides for tandem mass spectra analysis'
  source_content_hash: sha256:c2f08aa724902bd252ec7ea6fc4bed23219fe36b96f88d11234da9ce545e5863
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/privrja/MassSpecBlocks
  tools:
  - CycloBranch
  - MassSpecBlocks
  - PubChem
  - SmilesDrawer (modified)
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9333333333333333
n_claims: 15
---

# nrp-sequence-topology-classification

## Summary

Classify nonribosomal peptide/polyketide topology (linear, cyclic, branched, branch-cyclic) directly from the building-block graph produced by MassSpecBlocks' depth-first-search decomposition of a SMILES structure, and emit the sequence in CycloBranch notation.

## When to use

Use when a natural-product SMILES must be decomposed into ordered building blocks and its topology (cyclic vs branched vs branch-cyclic vs polyketide) must be determined for CycloBranch sequence databases.

## When NOT to use

- The goal is de novo interpretation of a tandem mass spectrum without a known structure — use CycloBranch/dereplication tools instead of structure-driven decomposition
- The input is a pre-existing building-block/sequence database rather than a SMILES structure to decompose
- Custom building-block metadata must come from a local container lookup first (Build Blocks searches the local database before regenerating properties)

## Inputs

- Chemical structure in SMILES format (retrieved from PubChem, ChemSpider, ChEBI, NP Atlas, COCONUT, or Norine, or edited in the MassSpecBlocks GUI)
- Molecular graph (array of vertices/atoms and array of edges/bonds) parsed from the SMILES

## Outputs

- Ordered building-block sequence in CycloBranch notation (N-terminus to C-terminus)
- Topology classification (linear, cyclic, branched, branch-cyclic, polyketide)
- Per-block residue molecular formulas and monoisotopic masses (e.g., written to pseudacyclin_a_blocks.csv)
- Plain-text export file usable by CycloBranch for mass spectra annotation

## How to apply

Parse the input SMILES into a molecular graph (vertices/edges, as in the modified SmilesDrawer library), then traverse the edge array to annotate peptide (and ester) bonds, skipping any bond whose resulting building block would contain three or fewer nonhydrogen atoms (e.g., the Ac-Ile bond in pseudacyclin A). Run a depth-first search over the annotated graph to generate building-block SMILES in order from N-terminus to C-terminus, fixing carboxyl groups at C-termini instead of formyl groups. Classify topology from DFS cues: a block found more than once marks the structure cyclic, a block with more than two neighbors marks it branched, and a block lacking a C-terminus marks it a polyketide. Emit the sequence in CycloBranch notation (e.g., [A]\[B]-[C]\[D]-[E] for a singly branched/branch-cyclic structure), and compute each residue's molecular formula and monoisotopic mass — normally by water elimination, but by elimination of two hydrogens for building blocks terminated by amines on both sides and lacking oxygen (e.g., Put: C4H12N2 molecular → C4H10N2 residue).

## Related tools

- **MassSpecBlocks** (Web-based tool that converts input SMILES structures into sequences of building blocks via bond annotation and depth-first search, and classifies topology) — https://github.com/privrja/MassSpecBlocks
- **SmilesDrawer (modified)** (Parses SMILES into the molecular graph (vertices and edges arrays) used for the DFS decomposition) — https://github.com/privrja/smilesDrawer
- **CycloBranch** (Consumes the exported plain-text sequence/building-block notation for annotating spectra of linear, cyclic, branched, and branch-cyclic NRPs) — https://ms.biomed.cas.cz/cyclobranch/docs/html/
- **PubChem** (Public database to retrieve the input structure by name, molecular formula, monoisotopic mass, identifier, or SMILES)

## Evaluation signals

- Pseudacyclin A must be classified as branch-cyclic and decomposed into six building blocks: cyclo(Pro-Ile-Ile-Orn-Phe) ring plus the Ac-Ile branch
- The Ac-Ile peptide bond must remain unmarked because the resulting Ac block would contain only three nonhydrogen atoms
- For diamine blocks lacking oxygen (e.g., putrescine), the residue formula must equal the molecular formula minus H2 (C4H12N2 → C4H10N2), not minus H2O
- Sequence output must be ordered N-terminus to C-terminus and in valid CycloBranch notation (e.g., [A]\[B]-[C]\[D]-[E])
- Exported plain text must load correctly as a building-block/sequence database in CycloBranch

## Limitations

- Bonds whose resulting block would contain three or fewer nonhydrogen atoms are not marked, so short substituents like Ac are skipped and may require manual handling
- The standard water-elimination residue-formula convention does not apply to building blocks terminated by amines on both sides and lacking oxygen (e.g., Put); a special two-hydrogen-elimination rule is needed
- Topology classification relies on DFS cues (repeat, neighbor count, C-terminus presence), which may not capture all PK/NRP structural conventions (e.g., reversed peptide-bond orientation in PK siderophores)

## Evidence

- [methods] DFS generates blocks and topology cues: "The depth-first search (DFS) was implemented to browse the molecular graph and generate the structures of building blocks in SMILES format."
- [results] Ac bond skipped due to small block size: "The peptide bond connecting Ac to Ile on the branch was not marked because the resulting block Ac would contain only three nonhydrogen atoms."
- [results] Pseudacyclin A validation result: "MassSpecBlocks correctly determined the branch-cyclic structure of the molecule."
- [abstract] SMILES-to-building-blocks purpose: "we introduce MassSpecBlocks - an open-source and web-based tool that converts the input chemical structures in SMILES format into sequences of building blocks"
- [abstract] Export target for sequences: "The sequences and building blocks created in MassSpecBlocks can be easily exported into a plain text format used by CycloBranch"
- [readme] Tool scope from README: "MassSpecBlocks: Database of Sequences and Building Blocks of Microbial Metabolites for Mass Spectra Analysis is an open-source web application"
