---
name: monoisotopic-mass-calculation
description: Use when you need theoretical monoisotopic masses for B/Y/internal glycan
  fragments of a new N-linked glycan not already in a database, to support MS1/MS2
  peak identification from LC-MS/MS data.
when_to_use_negative:
- The glycan is an O-linked glycan — the app's scope is explicitly limited to N-linked
  glycans
- The glycan is already present in the MS1/MS2 databases — the Fragment Generator
  is intended for new glycans not already located in the database
- A de novo glycan identification from spectra is required — the app explicitly lacks
  de novo identification capability and relies on database masses
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_0092
tools:
- name: GlyKAn AZ
  role: Hosts the Fragment Generator that parses the glycan structure into a MATLAB
    graph and emits fragment_table.csv with theoretical fragment masses
  repo: github.com/ZaylaSchaeffer/GlyKAn-AZ-application
- name: MATLAB
  role: Implementation language in which the glycan graph, depth-first-search fragment
    generation, and mass summation are executed
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1186/s12859-023-05346-5
    title: A MATLAB-based app to improve LC–MS/MS data analysis for N-linked glycan
      peak identification
  source_content_hash: sha256:403af66fa1baa816994abafc13925e760db8a492565e4711a3ec63926d9a43d6
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - GlyKAn AZ
  - MATLAB
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# monoisotopic-mass-calculation

## Summary

Computes theoretical monoisotopic fragment masses for glycan structures by summing residue masses from a monosaccharide mass table (e.g. Hex 180.0634, HexNAc 221.0899, Fuc 164.0685) with terminal-group corrections, as implemented in the GlyKAn AZ Fragment Generator for expanding glycan databases.

## When to use

Use when you need theoretical monoisotopic masses for B/Y/internal glycan fragments of a new N-linked glycan not already in a database, to support MS1/MS2 peak identification from LC-MS/MS data.

## When NOT to use

- The glycan is an O-linked glycan — the app's scope is explicitly limited to N-linked glycans
- The glycan is already present in the MS1/MS2 databases — the Fragment Generator is intended for new glycans not already located in the database
- A de novo glycan identification from spectra is required — the app explicitly lacks de novo identification capability and relies on database masses

## Inputs

- Glycan structure string (monosaccharide residues and glycosidic bond connectivity)
- Monosaccharide monoisotopic mass table (Xyl 150.0528, Fuc 164.0685, Hex 180.0634, HexN 221.0899, NAN 309.1060, etc.)
- MATLAB runtime environment

## Outputs

- fragment_table.csv listing fragment composition and theoretical monoisotopic mass for each B-, Y-, and internal ion
- Fragmentation patterns usable to expand the GlyKAn AZ MS1/MS2 glycan databases

## How to apply

Read the glycan structure string and the monosaccharide mass table (Xyl 150.0528, Fuc 164.0685, Hex 180.0634, HexN 221.0899, NAN 309.1060) in MATLAB, then parse the structure into a graph whose nodes are monosaccharide residues and whose edges are glycosidic bonds. Run a depth-first search over the graph that systematically cuts every combination of one to seven bonds, collecting the resulting connected subgraphs as candidate B-, Y-, and internal-ion fragments. For each fragment, compute the theoretical monoisotopic mass by summing residue masses from the table with appropriate terminal-group corrections. Write the fragment composition and theoretical mass to fragment_table.csv. QC rule: every emitted fragment mass must decompose exactly over the monosaccharide mass table entries (e.g. 2 Hex = 360.1268, Hex + HexNAc = 401.1533); flag any unreproducible mass as a QC failure.

## Related tools

- **GlyKAn AZ** (Hosts the Fragment Generator that parses the glycan structure into a MATLAB graph and emits fragment_table.csv with theoretical fragment masses) — github.com/ZaylaSchaeffer/GlyKAn-AZ-application
- **MATLAB** (Implementation language in which the glycan graph, depth-first-search fragment generation, and mass summation are executed)

## Evaluation signals

- Every emitted fragment mass decomposes exactly over the monosaccharide mass table — e.g. 2 Hex = 360.1268 and Hex + HexNAc = 401.1533 must be reproduced from table entries
- Any mass that cannot be reproduced from the table is flagged as a QC failure rather than silently emitted
- Generated fragment types are restricted to B-, Y-, and internal ions found via depth-first search cutting one to seven bonds
- Downstream MS1 search with these masses at a 10 ppm mass-accuracy threshold identifies known glycans as true positives (e.g. 20 TP / 0 FP at 25% relative intensity threshold)
- fragment_table.csv contains both fragment composition and theoretical mass for every candidate subgraph

## Limitations

- Fragment generation only covers cuts of one to seven bonds per glycan; larger or multi-cut fragmentation events beyond that range are not generated
- Masses must decompose exactly over the supplied monosaccharide mass table; residues absent from the table cause QC failures
- The scope covers N-linked glycans; O-linked glycans are not supported
- Fragmentation patterns for new glycans must be generated explicitly via the Fragment Generator — there is no de novo glycan identification

## Evidence

- [methods] Fragment Generator derives fragmentation patterns and theoretical fragment masses: [evidence span withheld — anchor status: not_found]
- [methods] Mass computation rule: [evidence span withheld — anchor status: not_found]
- [methods] QC validation rule: [evidence span withheld — anchor status: not_found]
- [abstract] Purpose of the Fragment Generator: "The databases provided with the app can be expanded through the Fragment Generator functionality which automatically identifies fragmentation patterns for new glycans"
- [readme] Tool context from repository: "Welcome to GlyKAn AZ, an application developed to streamline the N-linked glycan "
