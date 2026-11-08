---
name: cyclobranch-sequence-notation-encoding
description: Use when a decomposed NRP/PK building-block sequence must be written
  in CycloBranch plain-text notation — encoding ring-closing blocks, branch ordering,
  and terminal modifications — for mass spectra annotation or cross-tool comparison.
when_to_use_negative:
- The structure is multiply branched beyond a single branch point — it can only be
  encoded as type 'other' for CycloBranch compatibility, not as a standard branched
  notation
- You need a de novo spectrum annotation tool itself rather than a sequence encoding
  for CycloBranch input — MassSpecBlocks builds the database, CycloBranch does the
  annotation
- The monomeric structures have already been created by Smiles2Monomers or rBAN in
  a different export format that CycloBranch already accepts
tools:
- name: MassSpecBlocks
  role: Web tool whose Save/export step writes the constructed sequence and building
    blocks into the CycloBranch plain text format
  repo: github.com/privrja/MassSpecBlocks
- name: CycloBranch
  role: Target consumer of the exported plain-text sequence notation for de novo analysis
    and dereplication of tandem mass spectra
  repo: https://ms.biomed.cas.cz/cyclobranch/docs/html/
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1186/s13321-021-00530-2
    title: 'MassSpecBlocks: a web-based tool to create building blocks and sequences
      of nonribosomal peptides and polyketides for tandem mass spectra analysis'
  source_content_hash: sha256:53f98c87947ddbb7c3310d52ae0556c10a3e66da6312359963043b3661e93ca7
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CycloBranch
  - MassSpecBlocks
  - PubChem
  - SmilesDrawer
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# cyclobranch-sequence-notation-encoding

## Summary

Serializes a DFS-decomposed building-block sequence of an NRP or polyketide into the CycloBranch plain-text notation, encoding ring closures, branches, and terminal modifications so the sequence can annotate tandem mass spectra in CycloBranch.

## When to use

Use when a decomposed NRP/PK building-block sequence must be written in CycloBranch plain-text notation — encoding ring-closing blocks, branch ordering, and terminal modifications — for mass spectra annotation or cross-tool comparison.

## When NOT to use

- The structure is multiply branched beyond a single branch point — it can only be encoded as type 'other' for CycloBranch compatibility, not as a standard branched notation
- You need a de novo spectrum annotation tool itself rather than a sequence encoding for CycloBranch input — MassSpecBlocks builds the database, CycloBranch does the annotation
- The monomeric structures have already been created by Smiles2Monomers or rBAN in a different export format that CycloBranch already accepts

## Inputs

- Building-block SMILES strings from the depth-first search decomposition, ordered N-terminus to C-terminus
- Molecular graph (vertices = atoms, edges = bonds) parsed from the input SMILES via SmilesDrawer
- Structure-type call (linear, cyclic, branched, branch-cyclic, polyketide, other) derived from DFS properties
- Terminal modification formulas and monoisotopic mass offsets (e.g., C2H2O, +42.010565 Da)

## Outputs

- Sequence string in CycloBranch plain-text notation (e.g., [A]-[B]-[C]-[E] with E connected to A for cyclics)
- Terminal modifications listed separately from the main block sequence
- sequence_comparison.csv entries comparing constructed vs. article-reported sequences

## How to apply

After DFS decomposition yields building-block SMILES ordered from N-terminus to C-terminus, write the sequence as bracketed blocks in CycloBranch notation ([A]-[B]-[C]-[E]); for a cyclic structure place the ring-closing block last and mark its connection back to block A, and for a branched structure list branch blocks immediately after the branching block (e.g., Orn after the branching block in pseudacyclin A). Skip any bond whose resulting block would contain three or fewer nonhydrogen vertices — e.g., the Ac-to-Ile branch bond in pseudacyclin A is not marked because the Ac block would have only three nonhydrogen atoms — and record such terminal modifications separately with their monoisotopic mass offset (N-terminal C2H2O, +42.010565 Da). Fix carboxyl groups at C-termini instead of formyl groups when emitting block SMILES, and classify multiply branched structures as 'other' for CycloBranch compatibility. Export the finished sequence and blocks into the plain text format consumed by CycloBranch.

## Related tools

- **MassSpecBlocks** (Web tool whose Save/export step writes the constructed sequence and building blocks into the CycloBranch plain text format) — github.com/privrja/MassSpecBlocks
- **CycloBranch** (Target consumer of the exported plain-text sequence notation for de novo analysis and dereplication of tandem mass spectra) — https://ms.biomed.cas.cz/cyclobranch/docs/html/

## Evaluation signals

- The constructed sequence for pseudacyclin A reproduces six building blocks with the branch-cyclic type annotation (cyclo(Pro-Ile-Ile-Orn-Phe) ring plus Ac-Ile branch)
- The branch block (Orn) appears immediately after the branching block in the exported notation
- The Ac terminal modification is recorded separately with its monoisotopic mass offset +42.010565 Da rather than as a sixth sequence block
- Every block SMILES keeps carboxyl groups at C-termini instead of formyl groups
- Matches vs. article-reported CycloBranch-notation sequences are recorded in sequence_comparison.csv

## Limitations

- Bonds whose resulting block would contain three or fewer nonhydrogen atoms are not marked in the sequence, so terminal modifications like acetylation must be encoded separately
- Multiply branched structures can only be labeled 'other' in the structure-type call, limiting expressiveness of the CycloBranch notation for such molecules
- The Ac-to-Ile branch bond in pseudacyclin A is not automatically marked, so the branch must be recovered from the separately stored modification metadata

## Evidence

- [abstract] Export to CycloBranch plain text: "The sequences and building blocks created in MassSpecBlocks can be easily exported into a plain text format used by CycloBranch"
- [methods] DFS generates building blocks: "The depth-first search (DFS) was implemented to browse the molecular graph and generate the structures of building blocks in SMILES format."
- [results] Small-block bond skip rule: "The peptide bond connecting Ac to Ile on the branch was not marked because the resulting block Ac would contain only three nonhydrogen atoms."
- [results] Branch-cyclic call on pseudacyclin A: "MassSpecBlocks correctly determined the branch-cyclic structure of the molecule."
- [readme] README export purpose: "The application provides an export format for open source program [CycloBranch](https://ms.biomed.cas.cz/cyclobranch/docs/html/)"
