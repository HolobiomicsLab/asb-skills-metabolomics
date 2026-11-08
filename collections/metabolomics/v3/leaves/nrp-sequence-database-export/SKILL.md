---
name: nrp-sequence-database-export
description: Use when you need to feed a curated set of nonribosomal peptide or polyketide
  sequences and building blocks into CycloBranch for tandem mass spectra annotation
  or dereplication, and building blocks must first be assembled from SMILES in MassSpecBlocks.
when_to_use_negative:
- When the input is already a CycloBranch-compatible sequence database file — the
  export step is redundant and re-running MassSpecBlocks decomposition risks altering
  curated building-block boundaries.
- When the goal is to search public databases directly for structures rather than
  to construct a custom local sequence database for CycloBranch annotation.
- When working with tools that cannot accept custom NRP/PK sequence databases, such
  as NRPro (the paper notes 'users cannot create custom databases of building blocks
  and NRP sequences').
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0154
- http://edamontology.org/topic_3292
tools:
- name: MassSpecBlocks
  role: Web-based tool that converts SMILES chemical structures into building-block
    sequences and exports them as plain text for CycloBranch
  repo: github.com/privrja/MassSpecBlocks
- name: CycloBranch
  role: Downstream consumer of the exported plain-text database, used for de novo
    analysis and dereplication of linear, cyclic, branched, and branch-cyclic nonribosomal
    peptides
- name: smilesDrawer
  role: Used within MassSpecBlocks to draw chemical structures from SMILES format
    in the user-friendly graphical interface
  repo: github.com/privrja/smilesDrawer
- name: bbdgnc
  role: Previous application on which MassSpecBlocks is based
  repo: github.com/privrja/bbdgnc
provenance:
  source_task_ids:
  - task_001
  - task_004
  source_papers:
  - doi: 10.1186/s13321-021-00530-2
    title: 'MassSpecBlocks: a web-based tool to create building blocks and sequences
      of nonribosomal peptides and polyketides for tandem mass spectra analysis'
  source_content_hash: sha256:3b81a8d2039452e7b656fd5b1882a9c4892262dc2f30682bb08086b2b1031f06
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CycloBranch
  - MassSpecBlocks
  - bbdgnc
  - smilesDrawer
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_0154
  - http://edamontology.org/topic_3292
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7647058823529411
n_claims: 17
---

# nrp-sequence-database-export

## Summary

Export custom NRP/PK building-block and sequence databases from MassSpecBlocks into the plain-text format that CycloBranch ingests for dereplication and spectrum annotation, replacing manual construction of these databases.

## When to use

Use when you need to feed a curated set of nonribosomal peptide or polyketide sequences and building blocks into CycloBranch for tandem mass spectra annotation or dereplication, and building blocks must first be assembled from SMILES in MassSpecBlocks.

## When NOT to use

- When the input is already a CycloBranch-compatible sequence database file — the export step is redundant and re-running MassSpecBlocks decomposition risks altering curated building-block boundaries.
- When the goal is to search public databases directly for structures rather than to construct a custom local sequence database for CycloBranch annotation.
- When working with tools that cannot accept custom NRP/PK sequence databases, such as NRPro (the paper notes 'users cannot create custom databases of building blocks and NRP sequences').

## Inputs

- Chemical structures in SMILES format (NRP/PK compounds, e.g. branch-cyclic structures like pseudacyclin A)
- A MassSpecBlocks container selection (e.g. default 'Nonribosomal Peptides and Siderophores': 85 building blocks and 146 sequences)
- Building block properties (acronyms, residue formulas) either manually curated or retrieved from a local database container

## Outputs

- Plain-text export file containing lists of NRP/PK sequences, building blocks, and modifications in CycloBranch's accepted format
- A sequence database ready to be loaded into CycloBranch for dereplication of tandem mass spectra (e.g. 146 sequences against 44 CID and 44 HCD spectra)
- An annotation/dereplication report from CycloBranch matching the format shown in Additional file 1: Fig. S13

## How to apply

Build or import NRP/PK structures in MassSpecBlocks by searching molecular structures in SMILES format across PubChem, ChemSpider, ChEBI, NP Atlas, COCONUT, and Norine, then decompose them into building blocks (use the Build Blocks button, which searches SMILES strings first against a local container to retrieve block properties, falling back to RDKit-based generation). Correct the block boundaries where needed (e.g. 'Generic SMILES' to strip markup, 'Unique SMILES' to canonicalize via Weininger's CANON/GENES, and depth-first graph traversal to emit building-block SMILES), noting that peptide bonds yielding a block with only three non-hydrogen atoms are not automatically marked. Save the verified sequence, blocks, and metadata into the chosen container (the paper's default 'Nonribosomal Peptides and Siderophores' container holds 85 building blocks and 146 sequences). Export the container as plain text containing lists of sequences, blocks, and modifications — the format CycloBranch accepts — then load it in CycloBranch with parameters such as 5 ppm precursor and fragment ion m/z error tolerances, zero relative-peak-intensity threshold, and search restricted to b-ions and a-ions with at most one neutral loss of H2O or NH3. Validate the export by checking that the CycloBranch output report matches the report format shown in the paper's Additional file 1: Fig. S13.

## Related tools

- **MassSpecBlocks** (Web-based tool that converts SMILES chemical structures into building-block sequences and exports them as plain text for CycloBranch) — github.com/privrja/MassSpecBlocks
- **CycloBranch** (Downstream consumer of the exported plain-text database, used for de novo analysis and dereplication of linear, cyclic, branched, and branch-cyclic nonribosomal peptides)
- **smilesDrawer** (Used within MassSpecBlocks to draw chemical structures from SMILES format in the user-friendly graphical interface) — github.com/privrja/smilesDrawer
- **bbdgnc** (Previous application on which MassSpecBlocks is based) — github.com/privrja/bbdgnc

## Examples

```
python -c "import requests; r=requests.get('https://ms.biomed.cas.cz/msb/api/containers/1/export/plain'); open('nrp_sequences.txt','wb').write(r.content)"
```

## Evaluation signals

- Verify the export contains every sequence, building block, and modification present in the MassSpecBlocks container (e.g. the 85 building blocks / 146 sequences of the default container).
- Check that CycloBranch successfully loads the exported file without parsing errors and reproduces the report format of Additional file 1: Fig. S13.
- Confirm that annotation/dereplication accuracy computed on the exported database matches reference values (e.g. desferrioxamine B reported as single top hit with weighted ratio of matched peaks 96.4%).
- Validate that block decomposition does not drop marked peptide bonds incorrectly — bonds yielding a block of only three non-hydrogen atoms should be flagged for manual marking.
- Cross-check that exported building-block SMILES are canonical (Unique SMILES) and free of generic markups (Generic SMILES removed).

## Limitations

- The peptide bond connecting Ac to Ile on the branch was not automatically marked because the resulting Ac block would contain only three non-hydrogen atoms — manual correction of block boundaries may be required.
- Building block SMILES strings not present in the local database require RDKit-based generation of the remaining properties, which may yield less polished metadata.
- When exporting sequences from Norine, 'it's not always easy to export flawless sequences of building blocks for tandem mass spectra analysis.'
- The mass tolerance and ion-type configuration must be matched between MassSpecBlocks-exported databases and downstream tools (the paper unified settings with NRPro by restricting to b-ions and a-ions with at most one neutral loss of H2O or NH3).

## Evidence

- [abstract] Export to plain text format used by CycloBranch: "The sequences and building blocks created in MassSpecBlocks can be easily exported into a plain text format used by CycloBranch"
- [methods] Build Blocks generation via local database lookup: "A list of building blocks is generated using the button Build Blocks (Fig. 5). The SMILES strings of building blocks are searched in a local database first"
- [methods] Save sequence and blocks to container: "When a user checks the results, the sequence, blocks, and remaining metadata can be stored in the current container using the Save button."
- [results] Default container contents: [evidence span withheld — anchor status: not_found]
- [results] Peptide bond with fewer than four non-hydrogen atoms is not automatically marked: "The peptide bond connecting Ac to Ile on the branch was not marked because the resulting block Ac would contain only three nonhydrogen atoms."
- [intro] Export is easy from MassSpecBlocks: "The databases can be shared with collaborators and easily exported into a plain text format to be used for mass spectra analysis in CycloBranch software."
- [intro] NRPro cannot accept custom databases: "However, users cannot create custom databases of building blocks and NRP sequences."
- [readme] Export format purpose: "The application provides an export format for open source program CycloBranch"
- [results] Precursor mass filter demonstration: "In Additional file 1: Fig. S13, we show an output report if a precursor mass filter was enabled."
- [results] Unify configuration with NRPro for fair benchmarking: "we were looking only for b-ions, a-ions and allowed only one neutral loss of H2O or NH3."
