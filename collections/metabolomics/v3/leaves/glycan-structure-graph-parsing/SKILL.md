---
name: glycan-structure-graph-parsing
description: Use when a new N-linked glycan (e.g. A2G0M4F, A1G1F) must be added to
  the MS2 Fragment Database and its B/Y/internal fragmentation pattern and theoretical
  masses must be derived from its structure string.
when_to_use_negative:
- The glycan is already present in the provided MS1/MS2 Fragment Databases — the app's
  databases already cover it and regeneration is redundant
- The sample contains O-linked glycans — the app's scope is limited to N-linked glycans
- The goal is de novo glycan identification from spectra — the app explicitly lacks
  de novo capability and only identifies glycans in the database
tools:
- name: GlyKAn AZ
  role: Application whose Fragment Generator functionality performs the graph-based
    fragment enumeration to expand the glycan databases
  repo: github.com/ZaylaSchaeffer/GlyKAn-AZ-application
- name: MATLAB
  role: Runtime environment in which the glycan structure is parsed into a graph and
    the depth-first bond-cutting search is executed
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1186/s12859-023-05346-5
    title: A MATLAB-based app to improve LC–MS/MS data analysis for N-linked glycan
      peak identification
  source_content_hash: sha256:a28ebdfb308773216d71399ae77484b0612b7407c7f37ac4fbce49f47101d007
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - GlyKAn AZ
  - MATLAB
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# glycan-structure-graph-parsing

## Summary

Parses a glycan structure string into a MATLAB graph (nodes = monosaccharide residues, edges = glycosidic bonds) and enumerates B-, Y-, and internal-ion fragments via depth-first search by cutting every combination of one to seven bonds, yielding theoretical monoisotopic fragment masses for glycans absent from the GlyKAn AZ databases.

## When to use

Use when a new N-linked glycan (e.g. A2G0M4F, A1G1F) must be added to the MS2 Fragment Database and its B/Y/internal fragmentation pattern and theoretical masses must be derived from its structure string.

## When NOT to use

- The glycan is already present in the provided MS1/MS2 Fragment Databases — the app's databases already cover it and regeneration is redundant
- The sample contains O-linked glycans — the app's scope is limited to N-linked glycans
- The goal is de novo glycan identification from spectra — the app explicitly lacks de novo capability and only identifies glycans in the database

## Inputs

- Glycan structure string (monosaccharide residues and glycosidic bond topology)
- Monosaccharide monoisotopic mass table (Xyl 150.0528, Fuc 164.0685, Hex 180.0634, HexN 221.0899, NAN 309.1060, etc.)

## Outputs

- fragment_table.csv listing fragment composition and theoretical monoisotopic mass for B-, Y-, and internal ions
- Fragmentation pattern entries that expand the GlyKAn AZ MS2 Fragment Database for the new glycan

## How to apply

Read the glycan structure string together with the monosaccharide mass table (Xyl 150.0528, Fuc 164.0685, Hex 180.0634, HexN 221.0899, NAN 309.1060, etc.) in MATLAB, then parse the structure into a graph whose nodes are monosaccharide residues and whose edges are glycosidic bonds. Run a depth-first search over the graph that systematically cuts every combination of one to seven bonds and collects the resulting connected subgraphs as candidate fragments — classified as B-, Y-, or internal ions. Compute each fragment's theoretical monoisotopic mass by summing residue masses from the table with appropriate terminal-group corrections, and write fragment composition plus theoretical mass to fragment_table.csv. Validate every emitted mass by checking it decomposes exactly over the monosaccharide mass table entries (e.g. 2 Hex = 360.1268, Hex + HexNAc = 401.1533); flag any irreproducible mass as a QC failure. The resulting fragment database is what the GlyKAn AZ MS2 Peak Confirmation tab consumes to annotate LC–MS/MS spectra acquired on OrbiTrap or MALDI-TOF instruments.

## Related tools

- **GlyKAn AZ** (Application whose Fragment Generator functionality performs the graph-based fragment enumeration to expand the glycan databases) — github.com/ZaylaSchaeffer/GlyKAn-AZ-application
- **MATLAB** (Runtime environment in which the glycan structure is parsed into a graph and the depth-first bond-cutting search is executed)

## Evaluation signals

- Every emitted fragment mass decomposes exactly over the monosaccharide mass table entries (e.g. 2 Hex sums to 360.1268; Hex + HexNAc sums to 401.1533); any non-decomposable mass is a QC failure
- The fragment set covers bond-cut combinations from one through seven bonds per glycan, including B-, Y-, and internal ion types
- Theoretical masses written to fragment_table.csv reproduce the monosaccharide mass values in the table to full monoisotopic precision
- Fragment entries derived from the new glycan are loadable alongside the MS2 Fragment Database and MS2 fragment image repository in the MS2 Peak Confirmation tab

## Limitations

- The search enumerates fragments by cutting up to seven bonds per glycan, bounding the fragmentation space explored
- Fragment masses are only valid for monosaccharides represented in the mass table; residue types outside the table cannot be verified
- The app itself has no de novo glycan identification capability and is limited to N-linked glycans, so parsed structures must fall within that scope
- No software version for the GlyKAn AZ app is stated, and the repository has no changelog

## Evidence

- [methods] Graph representation and DFS fragment generation: [evidence span withheld — anchor status: not_found]
- [abstract] Fragment Generator expands databases: "The databases provided with the app can be expanded through the Fragment Generator functionality which automatically identifies fragmentation patterns for new glycans"
- [readme] App purpose from README: "Welcome to GlyKAn AZ, an application developed to streamline the N-linked glycan identification process from LC-MS/MS data."
