---
name: peptide-bond-graph-annotation
description: Use when an input SMILES of an NRP or PK siderophore (e.g. pseudacyclin
  A, desferrioxamine B retrieved from PubChem) must be decomposed into building-block
  sequences with topology classification for CycloBranch mass spectra annotation.
when_to_use_negative:
- The compound is non-peptidic — the article notes Dereplicator+ enables annotation
  of non-peptidic natural products, which is outside MassSpecBlocks' NRP/PK scope
- A bond splits off a block of three or fewer nonhydrogen atoms (e.g. the Ac-Ile bond)
  — that modification must be entered as a terminal modification rather than expected
  from automatic decomposition
- The goal is annotating tandem mass spectra rather than constructing sequence/block
  databases — export to CycloBranch and run the spectral search there (5 ppm precursor/fragment
  tolerance)
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0081
- http://edamontology.org/topic_3369
tools:
- name: MassSpecBlocks
  role: Performs the edge-array bond marking and DFS decomposition of the SMILES molecular
    graph into building-block sequences and exports them in CycloBranch format
  repo: github.com/privrja/MassSpecBlocks
- name: SmilesDrawer (privrja fork)
  role: Parses the input SMILES into the vertex/edge molecular graph representation
    used by the decomposition
  repo: https://github.com/privrja/smilesDrawer
- name: CycloBranch
  role: Consumes the exported plain-text sequence and building-block database for
    de novo analysis and dereplication of tandem mass spectra
  repo: https://ms.biomed.cas.cz/cyclobranch/docs/html/
- name: PubChem
  role: Source of input compound structures (e.g. pseudacyclin A, desferrioxamine
    B) retrievable by name, formula, mass, identifier, or SMILES
provenance:
  source_task_ids:
  - task_002
  - task_005
  source_papers:
  - doi: 10.1186/s13321-021-00530-2
    title: 'MassSpecBlocks: a web-based tool to create building blocks and sequences
      of nonribosomal peptides and polyketides for tandem mass spectra analysis'
  source_content_hash: sha256:e8833fb038869ca76520c8e4e393b09078cb3ad47295836b2f0dfa9d7e93c53a
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CycloBranch
  - MassSpecBlocks
  - PubChem
  - SmilesDrawer
  - SmilesDrawer (privrja fork)
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0081
  - http://edamontology.org/topic_3369
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# peptide-bond-graph-annotation

## Summary

Decomposes an input SMILES of a nonribosomal peptide or polyketide siderophore into its constituent building blocks by marking peptide/ester bonds on a molecular graph and running depth-first search, yielding a CycloBranch-compatible sequence and structure-type annotation (linear, cyclic, branched, branch-cyclic, polyketide).

## When to use

Use when an input SMILES of an NRP or PK siderophore (e.g. pseudacyclin A, desferrioxamine B retrieved from PubChem) must be decomposed into building-block sequences with topology classification for CycloBranch mass spectra annotation.

## When NOT to use

- The compound is non-peptidic — the article notes Dereplicator+ enables annotation of non-peptidic natural products, which is outside MassSpecBlocks' NRP/PK scope
- A bond splits off a block of three or fewer nonhydrogen atoms (e.g. the Ac-Ile bond) — that modification must be entered as a terminal modification rather than expected from automatic decomposition
- The goal is annotating tandem mass spectra rather than constructing sequence/block databases — export to CycloBranch and run the spectral search there (5 ppm precursor/fragment tolerance)

## Inputs

- SMILES string of the NRP/PK compound (searchable by name, molecular formula, monoisotopic mass, database identifier, or SMILES in PubChem, ChemSpider, ChEBI, NP Atlas, COCONUT, or Norine)
- Reference building-block sequence reported in source literature (for validation)
- Local building-block database (container) used to look up block SMILES and metadata

## Outputs

- Building-block SMILES strings ordered N-terminus to C-terminus
- Residue molecular formulas and monoisotopic masses per block
- Structure-type annotation (linear, cyclic, branched, branch-cyclic, polyketide, other)
- Sequence in CycloBranch plain-text notation with terminal modifications listed separately
- Comparison table (e.g. sequence_comparison.csv) of constructed vs. article-reported decomposition

## How to apply

Parse the SMILES into a molecular graph (vertices = atoms, edges = bonds) with the modified SmilesDrawer library, then traverse the edge array to mark peptide and ester bonds, skipping any bond whose resulting block would contain three or fewer nonhydrogen vertices (so terminal modifications like N-acetylation remain unmarked). Run depth-first search (DFS) over the annotated graph to generate building-block SMILES strings ordered from N-terminus to C-terminus, fixing carboxyl groups at C-termini instead of formyl groups. Classify topology from DFS properties: a recurring block marks the structure cyclic, a block with more than two neighbors marks it branched, both together mark branch-cyclic, and a block lacking a C-terminus marks it a polyketide ('other' for multiply branched structures). Compute residue formulas by water elimination, except for blocks terminated by amine groups on both sides and lacking oxygen (e.g. putrescine: C4H12N2 molecular formula gives C4H10N2 residue formula by elimination of two hydrogens). Emit the sequence in CycloBranch notation ([A]-[B]-[C]-[E] with E connected to A for cyclics; branch blocks placed after the branching block, e.g. Orn in pseudacyclin A) and record terminal modifications separately (e.g. N-terminal C2H2O, +42.010565 Da).

## Related tools

- **MassSpecBlocks** (Performs the edge-array bond marking and DFS decomposition of the SMILES molecular graph into building-block sequences and exports them in CycloBranch format) — github.com/privrja/MassSpecBlocks
- **SmilesDrawer (privrja fork)** (Parses the input SMILES into the vertex/edge molecular graph representation used by the decomposition) — https://github.com/privrja/smilesDrawer
- **CycloBranch** (Consumes the exported plain-text sequence and building-block database for de novo analysis and dereplication of tandem mass spectra) — https://ms.biomed.cas.cz/cyclobranch/docs/html/
- **PubChem** (Source of input compound structures (e.g. pseudacyclin A, desferrioxamine B) retrievable by name, formula, mass, identifier, or SMILES)

## Evaluation signals

- Pseudacyclin A decomposes into exactly six building blocks classified branch-cyclic: the cyclo(Pro-Ile-Ile-Orn-Phe) ring plus the Ac-Ile branch, matching the article-reported CycloBranch notation
- The Ac-Ile peptide bond remains unmarked because the Ac block would contain only three nonhydrogen atoms; acetylation appears as a separate N-terminal modification (+42.010565 Da, C2H2O)
- Putrescine residue formula is C4H10N2 (elimination of two hydrogens from C4H12N2), not a water-elimination product
- Desferrioxamine B decomposes into blocks including Hpd and Suc and is annotated as a linear polyketide (a block lacking a C-terminus)
- Cyclic sequences are written in CycloBranch notation with the final block (E) connected back to block A

## Limitations

- Bonds yielding blocks of three or fewer nonhydrogen atoms (terminal modifications like acetylation) are not automatically marked and must be entered separately
- The standard water-elimination residue formula is wrong for blocks terminated by amines on both sides without oxygen (e.g. Put), requiring the two-hydrogen-elimination convention
- Sequences with block counts greater than 20 may exceed HTTP request field limits in the predecessor application (LimitRequestFieldSize 16380 in httpd.conf)

## Evidence

- [abstract] SMILES-to-sequence conversion scope: "converts the input chemical structures in SMILES format into sequences of building blocks"
- [methods] DFS graph traversal for building-block generation: "The depth-first search (DFS) was implemented to browse the molecular graph and generate the structures of building blocks in SMILES format."
- [results] Minimum block-size rule for bond marking: "The peptide bond connecting Ac to Ile on the branch was not marked because the resulting block Ac would contain only three nonhydrogen atoms."
- [intro] PK siderophore building blocks complicate interpretation: "PK siderophores contain uncommon building blocks that turn over the orientation of peptide bonds in molecules"
- [readme] CycloBranch export target: "The application provides an export format for open source program [CycloBranch]"
