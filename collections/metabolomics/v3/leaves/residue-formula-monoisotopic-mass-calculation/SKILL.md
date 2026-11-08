---
name: residue-formula-monoisotopic-mass-calculation
description: Use when converting a SMILES structure into building-block sequences
  and each residue's formula/mass is needed for CycloBranch export, especially when
  blocks like Put (amine-terminated, oxygen-free) break the standard water-elimination
  convention.
when_to_use_negative:
- The input is already a building-block database with residue formulas (e.g. an existing
  CycloBranch or Norine export) — no recomputation is needed
- The target tool cannot accept custom building-block databases (e.g. NRPro, where
  users cannot create custom databases of building blocks and NRP sequences)
- The structure is a simple peptide composed solely of canonical amino acids whose
  residue formulas are already known
tools:
- name: MassSpecBlocks
  role: Web tool that parses SMILES into building-block sequences and computes block/residue
    formulas and monoisotopic masses via its Build Blocks step and DFS-based molecular-graph
    traversal
  repo: github.com/privrja/MassSpecBlocks
- name: SmilesDrawer (modified)
  role: Parses the SMILES string into the array-of-vertices/array-of-edges molecular
    graph used for bond annotation and DFS
  repo: https://github.com/privrja/smilesDrawer
- name: CycloBranch
  role: Consumer of the computed residue formulas, masses, and sequence exported in
    CycloBranch plain-text notation for tandem mass spectra annotation
  repo: https://ms.biomed.cas.cz/cyclobranch/docs/html/
- name: PubChem
  role: Public database from which the input SMILES structure (e.g. pseudacyclin A)
    is retrieved before formula computation
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1186/s13321-021-00530-2
    title: 'MassSpecBlocks: a web-based tool to create building blocks and sequences
      of nonribosomal peptides and polyketides for tandem mass spectra analysis'
  source_content_hash: sha256:edbb5cd0e008b43e4283298dab0a234d0dde672dfd3a3e44acff785e69b5ff26
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CycloBranch
  - MassSpecBlocks
  - PubChem
  - SmilesDrawer (modified)
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# residue-formula-monoisotopic-mass-calculation

## Summary

Computes residue molecular formulas and monoisotopic masses for nonribosomal peptide/polyketide building blocks during SMILES-to-sequence decomposition, applying water elimination by default and a two-hydrogen elimination for oxygen-free diamine blocks such as putrescine (Put: C4H12N2 -> C4H10N2).

## When to use

Use when converting a SMILES structure into building-block sequences and each residue's formula/mass is needed for CycloBranch export, especially when blocks like Put (amine-terminated, oxygen-free) break the standard water-elimination convention.

## When NOT to use

- The input is already a building-block database with residue formulas (e.g. an existing CycloBranch or Norine export) — no recomputation is needed
- The target tool cannot accept custom building-block databases (e.g. NRPro, where users cannot create custom databases of building blocks and NRP sequences)
- The structure is a simple peptide composed solely of canonical amino acids whose residue formulas are already known

## Inputs

- Chemical structure in SMILES format (e.g. pseudacyclin A retrieved from PubChem by name, formula, monoisotopic mass, identifier, or SMILES)
- Molecular graph of the structure (array of atom vertices and bond edges)
- Building-block SMILES strings (optionally matched against a local database container for metadata)

## Outputs

- Per-residue molecular formula (water-eliminated, or two-hydrogen-eliminated for diamine blocks like Put C4H10N2)
- Per-residue monoisotopic mass
- Sequence and building-block records in CycloBranch plain-text/CSV format (e.g. pseudacyclin_a_blocks.csv)

## How to apply

Parse the SMILES into a molecular graph of vertices (atoms) and edges (bonds), e.g. with the modified SmilesDrawer library, then annotate peptide and ester bonds while skipping any bond whose resulting block would contain three or fewer nonhydrogen vertices (so the Ac-Ile bond in pseudacyclin A stays unmarked). Run a depth-first search over the annotated graph to generate building-block SMILES from N-terminus to C-terminus, fixing carboxyl groups at C-termini. For each block, derive the residue formula by water elimination as the default; when the block is terminated by amine groups on both sides and contains no oxygen (e.g., Put), eliminate two hydrogens instead (C4H12N2 -> C4H10N2). Compute each residue's monoisotopic mass from the residue formula and write blocks, sequence, and structure type (e.g. branch-cyclic, CycloBranch notation [A]\[B]-[C]) to a CSV such as pseudacyclin_a_blocks.csv.

## Related tools

- **MassSpecBlocks** (Web tool that parses SMILES into building-block sequences and computes block/residue formulas and monoisotopic masses via its Build Blocks step and DFS-based molecular-graph traversal) — github.com/privrja/MassSpecBlocks
- **SmilesDrawer (modified)** (Parses the SMILES string into the array-of-vertices/array-of-edges molecular graph used for bond annotation and DFS) — https://github.com/privrja/smilesDrawer
- **CycloBranch** (Consumer of the computed residue formulas, masses, and sequence exported in CycloBranch plain-text notation for tandem mass spectra annotation) — https://ms.biomed.cas.cz/cyclobranch/docs/html/
- **PubChem** (Public database from which the input SMILES structure (e.g. pseudacyclin A) is retrieved before formula computation)

## Examples

```
python -c "from collections import Counter; f=Counter({'C':4,'H':12,'N':2}); f['H']-=2; print(''.join(f'{e}{n}' for e,n in sorted(f.items()) if n))"  # Put C4H12N2 -> C4H10N2
```

## Evaluation signals

- For an oxygen-free diamine block such as Put, the residue formula must equal the molecular formula minus H2 (C4H12N2 -> C4H10N2), not minus H2O
- For ordinary blocks, the residue formula must equal the block formula minus H2O (water elimination)
- Pseudacyclin A validation: the output must classify the molecule as branch-cyclic and decompose it into six building blocks — the cyclo(Pro-Ile-Ile-Orn-Phe) ring plus the Ac-Ile branch
- The bond joining Ac to Ile must remain unmarked because the Ac block would contain only three nonhydrogen atoms; no Ac residue should appear as a separate block if the skip rule fired
- The exported CSV (e.g. pseudacyclin_a_blocks.csv) must contain blocks, sequence, and structure type consistent with CycloBranch notation such as [A]\[B]-[C]\[D]-[E] for singly branched/branch-cyclic

## Limitations

- The two-hydrogen elimination applies only to blocks terminated by amine groups on both sides and lacking oxygen; applying water elimination to such blocks (like Put) yields a wrong residue formula
- Peptide/ester bonds producing blocks with three or fewer nonhydrogen vertices are not marked, so very small moieties (e.g. Ac) are skipped and may need manual handling
- Residue formulas are searched against a local database container first; blocks absent from the local database rely on the computed conventions and may lack remaining metadata

## Evidence

- [other] Water elimination default and Put exception: [evidence span withheld — anchor status: not_found]
- [other] Put formulas: [evidence span withheld — anchor status: not_found]
- [methods] DFS block generation: "The depth-first search (DFS) was implemented to browse the molecular graph and generate the structures of building blocks in SMILES format."
- [results] Ac bond skip rule: "The peptide bond connecting Ac to Ile on the branch was not marked because the resulting block Ac would contain only three nonhydrogen atoms."
- [methods] Local database lookup for block metadata: "The SMILES strings of building blocks are searched in a local database first (i.e., in a selected container) to obtain the remaining properties of the blocks."
