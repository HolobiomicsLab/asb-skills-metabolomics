---
name: smiles-molecular-graph-parsing
description: Use when an input chemical structure in SMILES format (e.g., from PubChem
  or ChemSpider) must be converted into a building-block sequence in CycloBranch notation
  for tandem mass spectra analysis of NRPs or PK siderophores.
when_to_use_negative:
- The input is already a tandem mass spectrum or peak list — use CycloBranch/Dereplicator
  annotation instead of structure decomposition
- The goal is de novo spectral sequencing without a known input SMILES structure
- The building-block SMILES strings already exist in the local database with full
  metadata, so re-parsing the structure is unnecessary
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_2258
tools:
- name: MassSpecBlocks
  role: Web-based tool that parses input SMILES structures and converts them into
    sequences of building blocks with structure-type classification and CycloBranch
    export
  repo: https://github.com/privrja/MassSpecBlocks
- name: SmilesDrawer
  role: Modified library used to parse SMILES into the vertex/edge molecular graph
    that the DFS decomposition traverses
  repo: https://github.com/privrja/smilesDrawer
- name: CycloBranch
  role: Downstream consumer of the exported plain-text sequences and building blocks
    for mass spectra annotation
provenance:
  source_task_ids:
  - task_002
  - task_005
  source_papers:
  - doi: 10.1186/s13321-021-00530-2
    title: 'MassSpecBlocks: a web-based tool to create building blocks and sequences
      of nonribosomal peptides and polyketides for tandem mass spectra analysis'
  source_content_hash: sha256:d736720951e56d71f822df7c11766877e86723063b3be243532ad498fe0710e5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/privrja/MassSpecBlocks
  tools:
  - CycloBranch
  - MassSpecBlocks
  - PubChem
  - SmilesDrawer
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_2258
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 14
---

# smiles-molecular-graph-parsing

## Summary

Parse input SMILES of NRPs and polyketides into a molecular graph (atom vertices, bond edges) and decompose it into ordered building-block sequences via annotated bond marking and depth-first search, as implemented in MassSpecBlocks.

## When to use

Use when an input chemical structure in SMILES format (e.g., from PubChem or ChemSpider) must be converted into a building-block sequence in CycloBranch notation for tandem mass spectra analysis of NRPs or PK siderophores.

## When NOT to use

- The input is already a tandem mass spectrum or peak list — use CycloBranch/Dereplicator annotation instead of structure decomposition
- The goal is de novo spectral sequencing without a known input SMILES structure
- The building-block SMILES strings already exist in the local database with full metadata, so re-parsing the structure is unnecessary

## Inputs

- SMILES string of a natural product (NRP or polyketide), e.g., pseudacyclin A or desferrioxamine B retrieved from PubChem
- Molecular graph (array of atom vertices and bond edges) as produced by the modified SmilesDrawer parser
- Optional local database/container of known building-block SMILES with metadata (acronym, formula, mass)

## Outputs

- Building-block SMILES strings ordered from N-terminus to C-terminus
- Residue molecular formulas and monoisotopic masses per building block (e.g., C4H10N2 for Putrescine)
- Structure-type annotation (linear, cyclic, branched, branch-cyclic, polyketide, other)
- Sequence in CycloBranch notation exportable to plain text (e.g., pseudacyclin_a_blocks.csv, sequence_comparison.csv)

## How to apply

Parse the SMILES into a molecular graph of vertices (atoms) and edges (bonds) using the modified SmilesDrawer library. Traverse the edge array to mark peptide and ester bonds, skipping any bond whose resulting block would contain three or fewer nonhydrogen vertices (e.g., the Ac-Ile bond, since the Ac block has only three nonhydrogen atoms). Run a depth-first search (DFS) over the annotated graph to generate building-block SMILES strings ordered from N-terminus to C-terminus, fixing carboxyl groups at C-termini instead of formyl groups. Classify topology from DFS cues: a block found more than once is cyclic, a block with more than two neighbors is branched, a block lacking a C-terminus marks a polyketide. Compute residue formulas and monoisotopic masses per block — normally by water elimination, but for blocks terminated by amine groups on both sides and lacking oxygen (e.g., Putrescine, C4H12N2 molecular formula) eliminate two hydrogens instead (C4H10N2). Emit the sequence in CycloBranch notation (e.g., [A]-[B]-[C]-[E] with E connected to A for cyclics) and store blocks, sequence, and structure type.

## Related tools

- **MassSpecBlocks** (Web-based tool that parses input SMILES structures and converts them into sequences of building blocks with structure-type classification and CycloBranch export) — https://github.com/privrja/MassSpecBlocks
- **SmilesDrawer** (Modified library used to parse SMILES into the vertex/edge molecular graph that the DFS decomposition traverses) — https://github.com/privrja/smilesDrawer
- **CycloBranch** (Downstream consumer of the exported plain-text sequences and building blocks for mass spectra annotation)

## Examples

```
SmilesDrawer.parse(input.value, function(tree) { smilesDrawer.draw(tree, "example-canvas", "light", false); });
```

## Evaluation signals

- Pseudacyclin A must decompose into six building blocks classified as branch-cyclic: cyclo(Pro-Ile-Ile-Orn-Phe) ring plus the Ac-Ile branch
- The Ac-Ile bond must remain unmarked because the Ac block would contain only three nonhydrogen vertices
- Building blocks without a double-sided amine termination must yield residue formulas via water elimination; Putrescine must yield C4H10N2 residue formula from C4H12N2 molecular formula
- Desferrioxamine B must be decomposed into building blocks Hpd and Suc and annotated as a linear polyketide
- The constructed sequence must be reproducible in CycloBranch plain-text notation and comparable against article-reported sequences in sequence_comparison.csv

## Limitations

- Peptide/ester bonds are skipped when the resulting block would contain three or fewer nonhydrogen vertices, so terminal modifications like acetylation do not form independent blocks
- The standard water-elimination residue-formula convention does not apply to blocks terminated by amine groups on both sides and lacking oxygen, requiring a special two-hydrogen elimination rule
- Decomposition is graph-based and may mis-classify multiply branched structures, which are emitted as 'other' for CycloBranch compatibility

## Evidence

- [methods] DFS generates building-block SMILES: "The depth-first search (DFS) was implemented to browse the molecular graph and generate the structures of building blocks in SMILES format."
- [results] Ac block skipped due to size: "The peptide bond connecting Ac to Ile on the branch was not marked because the resulting block Ac would contain only three nonhydrogen atoms."
- [abstract] Core conversion of SMILES to sequences: "Here, we introduce MassSpecBlocks - an open-source and web-based tool that converts the input chemical structures in SMILES format into sequences of building blocks"
- [readme] SmilesDrawer parsing API: "Clean the input (remove unrecognized characters, such as spaces and tabs) and parse it
          SmilesDrawer.parse(input.value, function(tree) {"
- [methods] Building block generation entry point: "A list of building blocks is generated using the button Build Blocks (Fig. 5)."
