---
name: sequence-reference-comparison-evaluation
description: Use when a tool has decomposed an NRP or linear PK siderophore SMILES
  into building blocks and you must confirm the sequence, branch, and structure-type
  annotation against a published reference decomposition.
when_to_use_negative:
- The goal is tandem mass spectra annotation/dereplication itself (b-ion/a-ion searches
  with 5 ppm tolerance in CycloBranch/NRPro or 0.005 Da in Dereplicator) rather than
  verifying a sequence construction — this skill only evaluates sequence correctness,
  not spectral matching.
- The input is already a sequence in CycloBranch plain-text format rather than a SMILES
  structure needing decomposition.
- The compound contains blocks smaller than the three-nonhydrogen-vertex cutoff whose
  correctness must be independently asserted (e.g. verifying a free Ac block), since
  such bonds are intentionally not marked.
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: MassSpecBlocks
  role: Runs the DFS decomposition of input SMILES into building-block sequences and
    exports them in CycloBranch notation for comparison against reference sequences
  repo: github.com/privrja/MassSpecBlocks
- name: SmilesDrawer
  role: Parses SMILES into the molecular graph (vertices = atoms, edges = bonds) that
    the DFS decomposition traverses
  repo: https://github.com/privrja/smilesDrawer
- name: PubChem
  role: Source of the input compound structures, searched by name, formula, monoisotopic
    mass, database identifier, or SMILES
- name: CycloBranch
  role: Defines the target plain-text sequence notation ([A]-[B]-[C]-[E]) into which
    constructed sequences are exported for the reference comparison
  repo: https://ms.biomed.cas.cz/cyclobranch/docs/html/
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1186/s13321-021-00530-2
    title: 'MassSpecBlocks: a web-based tool to create building blocks and sequences
      of nonribosomal peptides and polyketides for tandem mass spectra analysis'
  source_content_hash: sha256:89975da85642b4c4291cf152de5355544fb536cf60aa2317ceb1b27b3abd8b1c
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CycloBranch
  - MassSpecBlocks
  - PubChem
  - SmilesDrawer
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# sequence-reference-comparison-evaluation

## Summary

Validates SMILES-to-building-block decomposition (e.g. via MassSpecBlocks' DFS on a molecular graph) by comparing the constructed sequence and structure-type call against article-reported reference sequences written in CycloBranch notation, recording matches (e.g. in sequence_comparison.csv).

## When to use

Use when a tool has decomposed an NRP or linear PK siderophore SMILES into building blocks and you must confirm the sequence, branch, and structure-type annotation against a published reference decomposition.

## When NOT to use

- The goal is tandem mass spectra annotation/dereplication itself (b-ion/a-ion searches with 5 ppm tolerance in CycloBranch/NRPro or 0.005 Da in Dereplicator) rather than verifying a sequence construction — this skill only evaluates sequence correctness, not spectral matching.
- The input is already a sequence in CycloBranch plain-text format rather than a SMILES structure needing decomposition.
- The compound contains blocks smaller than the three-nonhydrogen-vertex cutoff whose correctness must be independently asserted (e.g. verifying a free Ac block), since such bonds are intentionally not marked.

## Inputs

- Input compound SMILES strings (e.g. pseudacyclin A, desferrioxamine B, retrievable from PubChem by database identifier)
- Article-reported reference building-block sequences and structure-type calls
- Molecular graph parsed by SmilesDrawer (vertices = atoms, edges = bonds)

## Outputs

- Constructed building-block sequence in CycloBranch notation ([A]-[B]-[C]-[E] with E connected to A for cyclics)
- Structure-type classification (cyclic, branched, branch-cyclic, linear polyketide, other)
- sequence_comparison.csv recording match/mismatch against each reference sequence

## How to apply

Load the input SMILES (retrieved from PubChem by database identifier) and the article-reported reference sequence, then run or reproduce the decomposition: parse the SMILES into a molecular graph with SmilesDrawer (vertices = atoms, edges = bonds), mark peptide/ester bonds while skipping any bond whose resulting block would contain three or fewer nonhydrogen vertices (so terminal modifications like Ac stay attached, not independent blocks), and run the depth-first search to emit building-block SMILES ordered N-terminus to C-terminus with carboxyl groups fixed at C-termini. Classify structure type from DFS properties — cyclic if a block recurs, branched if a block has >2 neighbors, branch-cyclic if both; polyketide if a block lacks a C-terminus, 'other' for multiply branched structures. Write the sequence in CycloBranch notation ([A]-[B]-[C]-[E], E connected to A for cyclics; branch blocks after the branching block, e.g. Orn in pseudacyclin A) with terminal modifications listed separately (e.g. N-terminal C2H2O, +42.010565 Da). Accept the output only when the constructed sequence and type call reproduce the reference (pseudacyclin A: branch-cyclic, six blocks — cyclo(Pro-Ile-Ile-Orn-Phe) ring plus Ac-Ile branch; desferrioxamine B: Hpd and Suc blocks, linear polyketide), logging each comparison in sequence_comparison.csv.

## Related tools

- **MassSpecBlocks** (Runs the DFS decomposition of input SMILES into building-block sequences and exports them in CycloBranch notation for comparison against reference sequences) — github.com/privrja/MassSpecBlocks
- **SmilesDrawer** (Parses SMILES into the molecular graph (vertices = atoms, edges = bonds) that the DFS decomposition traverses) — https://github.com/privrja/smilesDrawer
- **PubChem** (Source of the input compound structures, searched by name, formula, monoisotopic mass, database identifier, or SMILES)
- **CycloBranch** (Defines the target plain-text sequence notation ([A]-[B]-[C]-[E]) into which constructed sequences are exported for the reference comparison) — https://ms.biomed.cas.cz/cyclobranch/docs/html/

## Examples

```
cd thesis && symfony server:start  # MassSpecBlocks REST API at localhost:8000/rest, docs at /rest/doc
```

## Evaluation signals

- Constructed sequence for pseudacyclin A decomposes into exactly six building blocks matching the reference cyclo(Pro-Ile-Ile-Orn-Phe) ring plus Ac-Ile branch, annotated branch-cyclic.
- Desferrioxamine B decomposes to the reported Hpd and Suc blocks with a linear polyketide structure-type call.
- Terminal modifications are reported separately with correct mass shifts (e.g. N-terminal C2H2O, +42.010565 Da) rather than as independent blocks.
- Every constructed sequence vs. reference comparison is recorded in sequence_comparison.csv with an explicit match/mismatch verdict.
- Sequences serialize losslessly into CycloBranch plain-text format (rings as [A]-[B]-[C]-[E] with E connected to A; branch blocks placed after the branching block).

## Limitations

- Bonds whose resulting block would contain only three or fewer nonhydrogen atoms are not automatically marked, so very small terminal blocks (e.g. Ac) cannot be validated as independent blocks.
- Multiply branched structures are classified only as 'other' for CycloBranch compatibility, limiting type resolution.
- Automated interpretation of NRP tandem mass spectra remains challenging because of complex molecular structures, and PK siderophores contain uncommon building blocks that reverse peptide bond orientation (e.g. desferrioxamines), complicating verification.

## Evidence

- [methods] The depth-first search (DFS) was implemented to browse the molecular graph and generate the structures of building blocks in SMILES format.: "The depth-first search (DFS) was implemented to browse the molecular graph and generate the structures of building blocks in SMILES format."
- [results] Ac block cutoff behavior: "The peptide bond connecting Ac to Ile on the branch was not marked because the resulting block Ac would contain only three nonhydrogen atoms."
- [results] Correct decomposition of showcased compound: "MassSpecBlocks correctly determined the branch-cyclic structure of the molecule."
- [abstract] Core conversion function of the tool: "converts the input chemical structures in SMILES format into sequences of building blocks"
- [abstract] Export format for comparison: "The sequences and building blocks created in MassSpecBlocks can be easily exported into a plain text format used by CycloBranch"
- [readme] REST API for programmatic access: "Documentation of API is [localhost:8000/rest/doc](https://localhost:8000/rest/doc)
Rest API is on [localhost:8000/rest]"
