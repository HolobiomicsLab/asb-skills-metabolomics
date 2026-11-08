---
name: nrp-block-decomposition-via-depth-first-search
description: Use when you have a NRP or linear PK siderophore SMILES (e.g. pseudacyclin
  A, desferrioxamine B) and need its constituent building-block sequence and structure
  type (linear/cyclic/branched/branch-cyclic/polyketide) for mass-spectra annotation
  in CycloBranch.
when_to_use_negative:
- Input is raw tandem mass spectra rather than SMILES structures — use CycloBranch,
  NRPro, Dereplicator/Dereplicator+ for spectra annotation instead
- The target compounds are non-peptidic natural products (Dereplicator+ territory),
  not NRP/PK structures decomposable into peptide/ester-bond building blocks
- The goal is de novo spectral annotation; MassSpecBlocks only builds sequence and
  building-block databases, it does not annotate spectra
edam_topics:
- http://edamontology.org/topic_3369
- http://edamontology.org/topic_0091
tools:
- name: MassSpecBlocks
  role: Web-based tool (open-source, offline-installable, REST API) that converts
    input SMILES into building-block sequences and exports them in CycloBranch plain-text
    format; implements the Build Blocks/DFS workflow
  repo: https://github.com/privrja/MassSpecBlocks
- name: SmilesDrawer
  role: Parses input SMILES into the molecular graph (vertices = atoms, edges = bonds)
    that the DFS decomposition traverses
  repo: https://github.com/privrja/smilesDrawer
- name: CycloBranch
  role: Target consumer of the exported sequence/building-block plain-text databases
    for annotating linear, cyclic, branched, and branch-cyclic NRP tandem mass spectra
  repo: https://ms.biomed.cas.cz/cyclobranch/docs/html/
- name: PubChem
  role: Source of input compound SMILES retrieved by name, molecular formula, monoisotopic
    mass, or database identifier
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1186/s13321-021-00530-2
    title: 'MassSpecBlocks: a web-based tool to create building blocks and sequences
      of nonribosomal peptides and polyketides for tandem mass spectra analysis'
  source_content_hash: sha256:96c40eab2a744a36f6d7d0bfe2826fa78de4be39a042c17eb0a4013fbd3e3a33
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/privrja/MassSpecBlocks
  tools:
  - CycloBranch
  - MassSpecBlocks
  - PubChem
  - SmilesDrawer
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3369
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# NRP/PK building-block decomposition from SMILES via depth-first search

## Summary

Reproduces MassSpecBlocks-style decomposition of an input SMILES into ordered building-block SMILES sequences for nonribosomal peptides and polyketides, using SmilesDrawer molecular-graph parsing and a depth-first search with CycloBranch-compatible sequence notation.

## When to use

Use when you have a NRP or linear PK siderophore SMILES (e.g. pseudacyclin A, desferrioxamine B) and need its constituent building-block sequence and structure type (linear/cyclic/branched/branch-cyclic/polyketide) for mass-spectra annotation in CycloBranch.

## When NOT to use

- Input is raw tandem mass spectra rather than SMILES structures — use CycloBranch, NRPro, Dereplicator/Dereplicator+ for spectra annotation instead
- The target compounds are non-peptidic natural products (Dereplicator+ territory), not NRP/PK structures decomposable into peptide/ester-bond building blocks
- The goal is de novo spectral annotation; MassSpecBlocks only builds sequence and building-block databases, it does not annotate spectra

## Inputs

- Input chemical structure in SMILES format (e.g. pseudacyclin A or desferrioxamine B retrieved from PubChem, ChemSpider, ChEBI, NP Atlas, COCONUT, or Norine by name, formula, monoisotopic mass, database identifier, or SMILES)
- Reference building-block sequences reported in a source article (for validation)
- Local database/container of building-block SMILES and metadata

## Outputs

- Ordered building-block SMILES strings from N-terminus to C-terminus
- Structure-type annotation: linear, cyclic, branched, branch-cyclic, polyketide, or other
- Constructed sequence in CycloBranch plain-text notation with terminal modifications listed separately
- sequence_comparison.csv recording matches against reference decompositions

## How to apply

Parse the input SMILES into a molecular graph (vertices = atoms, edges = bonds) with SmilesDrawer and traverse the edge array to mark peptide/ester bonds, skipping any bond whose resulting block would contain three or fewer nonhydrogen vertices so terminal modifications such as acetylation do not form independent blocks. Run a depth-first search (DFS) over the molecular graph to generate building-block SMILES strings ordered from N-terminus to C-terminus, fixing carboxyl groups at C-termini instead of formyl groups. Classify structure type from DFS properties: a recurring block marks cyclic, a block with more than 2 neighbors marks branched, both together mark branch-cyclic, a block lacking a C-terminus marks polyketide, and multiply branched structures are marked 'other' for CycloBranch compatibility. Write the constructed sequence in CycloBranch notation ([A]-[B]-[C]-[E] with E connected to A for cyclics, branch blocks after the branching block, e.g. Orn in pseudacyclin A), listing terminal modifications separately (e.g. N-terminal C2H2O, +42.010565 Da). Search generated block SMILES against a local database first to fill in remaining block metadata before saving.

## Related tools

- **MassSpecBlocks** (Web-based tool (open-source, offline-installable, REST API) that converts input SMILES into building-block sequences and exports them in CycloBranch plain-text format; implements the Build Blocks/DFS workflow) — https://github.com/privrja/MassSpecBlocks
- **SmilesDrawer** (Parses input SMILES into the molecular graph (vertices = atoms, edges = bonds) that the DFS decomposition traverses) — https://github.com/privrja/smilesDrawer
- **CycloBranch** (Target consumer of the exported sequence/building-block plain-text databases for annotating linear, cyclic, branched, and branch-cyclic NRP tandem mass spectra) — https://ms.biomed.cas.cz/cyclobranch/docs/html/
- **PubChem** (Source of input compound SMILES retrieved by name, molecular formula, monoisotopic mass, or database identifier)

## Evaluation signals

- Constructed pseudacyclin A sequence reproduces the article-reported decomposition into six building blocks (cyclo(Pro-Ile-Ile-Orn-Phe) ring plus Ac-Ile branch) with the branch-cyclic type annotation in CycloBranch notation
- Desferrioxamine B decomposes into its Hpd and Suc building blocks and is annotated as a linear polyketide
- Bonds whose resulting block would contain three or fewer nonhydrogen vertices are not marked (e.g. the Ac-to-Ile peptide bond on pseudacyclin A's branch)
- Block recurrence (cyclic), block degree > 2 (branched), and missing C-terminus (polyketide) classifications match the reference type calls
- Per-compound match records are written to sequence_comparison.csv and agree with article-reported CycloBranch-notation sequences

## Limitations

- Peptide bonds connecting very small terminal modifications are not marked because the resulting block would contain only three or fewer nonhydrogen atoms (e.g. the Ac block in pseudacyclin A)
- PK siderophores such as desferrioxamines and ornibactins contain uncommon building blocks that turn over the orientation of peptide bonds, complicating interpretation
- Building-block SMILES are searched in a local database first; blocks not present there require additional handling for metadata (acronyms, formulas)
- Automated interpretation of NRP tandem mass spectra remains challenging due to the complex structures of the molecules — this skill builds sequences, it does not guarantee correct spectral annotation

## Evidence

- [methods] DFS generates building-block SMILES: "The depth-first search (DFS) was implemented to browse the molecular graph and generate the structures of building blocks in SMILES format."
- [results] Small blocks are not marked: "The peptide bond connecting Ac to Ile on the branch was not marked because the resulting block Ac would contain only three nonhydrogen atoms."
- [abstract] SMILES to building-block sequences: "an open-source and web-based tool that converts the input chemical structures in SMILES format into sequences of building blocks"
- [methods] Build Blocks button: "A list of building blocks is generated using the button Build Blocks (Fig. 5). The SMILES strings of building blocks are searched in a local database first"
- [results] Correct branch-cyclic call: "MassSpecBlocks correctly determined the branch-cyclic structure of the molecule."
- [readme] Tool scope from README: "MassSpecBlocks: Database of Sequences and Building Blocks of Microbial Metabolites for Mass Spectra Analysis is an open-source web application"
