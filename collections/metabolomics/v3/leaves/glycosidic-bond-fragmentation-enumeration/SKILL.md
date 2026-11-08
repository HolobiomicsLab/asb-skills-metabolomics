---
name: glycosidic-bond-fragmentation-enumeration
description: Use when a glycan of interest is not present in the GlyKAn AZ MS2 Fragment
  Database and its theoretical fragmentation pattern (B/Y/internal ions) and monoisotopic
  fragment masses must be derived from its structure string.
when_to_use_negative:
- The glycan is already present in the GlyKAn AZ MS2 Fragment Database — use the existing
  database entries rather than regenerating fragments
- The analysis target is O-linked glycans, which the app's scope does not currently
  cover
- A de novo glycan identification is needed — the skill only derives fragments for
  structures you can already specify, it cannot discover unknown glycans
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: GlyKAn AZ
  role: Hosts the Fragment Generator functionality that parses the glycan graph, enumerates
    1–7 bond cuts, and emits fragment masses to expand the MS2 Fragment Database
  repo: github.com/ZaylaSchaeffer/GlyKAn-AZ-application
- name: MATLAB
  role: Implementation language/runtime in which the glycan graph, depth-first search
    over bond cuts, and mass computation are executed
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1186/s12859-023-05346-5
    title: A MATLAB-based app to improve LC–MS/MS data analysis for N-linked glycan
      peak identification
  source_content_hash: sha256:d9b540b450358c47f1d09c3926afc8dbaf6ca91d735362067a24c64edd3cc22b
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - GlyKAn AZ
  - MATLAB
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# glycosidic-bond-fragmentation-enumeration

## Summary

Enumerates theoretical B-, Y-, and internal-ion fragments for a new N-linked glycan by representing it as a MATLAB graph (nodes = monosaccharides, edges = glycosidic bonds) and cutting every combination of one to seven bonds, producing the fragment masses needed to expand the GlyKAn AZ MS2 Fragment Database.

## When to use

Use when a glycan of interest is not present in the GlyKAn AZ MS2 Fragment Database and its theoretical fragmentation pattern (B/Y/internal ions) and monoisotopic fragment masses must be derived from its structure string.

## When NOT to use

- The glycan is already present in the GlyKAn AZ MS2 Fragment Database — use the existing database entries rather than regenerating fragments
- The analysis target is O-linked glycans, which the app's scope does not currently cover
- A de novo glycan identification is needed — the skill only derives fragments for structures you can already specify, it cannot discover unknown glycans

## Inputs

- Glycan structure string (monosaccharide residues and their glycosidic bond connectivity)
- Monosaccharide mass table (e.g. Xyl 150.0528, Fuc 164.0685, Hex 180.0634, HexN 221.0899, NAN 309.1060)
- MATLAB environment running the GlyKAn AZ Fragment Generator

## Outputs

- fragment_table.csv listing each fragment's composition and theoretical monoisotopic mass
- Fragmentation pattern (B/Y/internal ions) for insertion into the GlyKAn AZ MS2 Fragment Database

## How to apply

Read the glycan structure string and the monosaccharide mass table (Xyl 150.0528, Fuc 164.0685, Hex 180.0634, HexN 221.0899, NAN 309.1060) in MATLAB, then parse the structure string into a graph whose nodes are monosaccharide residues and whose edges are glycosidic bonds. Perform a depth-first search over the graph that systematically cuts every combination of one to seven bonds, collecting the resulting connected subgraphs as candidate B-, Y-, and internal-ion fragments. Compute each fragment's theoretical monoisotopic mass by summing residue masses from the monosaccharide mass table with appropriate terminal-group corrections. Write the fragment list (composition and theoretical mass) to fragment_table.csv. Validate by checking that every emitted fragment mass decomposes exactly over the monosaccharide mass table entries (e.g. 2 Hex = 360.1268, Hex + HexNAc = 401.1533); flag any mass that cannot be reproduced from the table as a QC failure.

## Related tools

- **GlyKAn AZ** (Hosts the Fragment Generator functionality that parses the glycan graph, enumerates 1–7 bond cuts, and emits fragment masses to expand the MS2 Fragment Database) — github.com/ZaylaSchaeffer/GlyKAn-AZ-application
- **MATLAB** (Implementation language/runtime in which the glycan graph, depth-first search over bond cuts, and mass computation are executed)

## Evaluation signals

- Every emitted fragment mass decomposes exactly over the monosaccharide mass table entries; any mass that cannot be reproduced from the table is a QC failure
- Spot-check sums reproduce known values: 2 Hex = 360.1268 and Hex + HexNAc = 401.1533
- Every candidate fragment is a connected subgraph obtained by cutting between one and seven glycosidic bonds
- fragment_table.csv contains both fragment composition and theoretical mass for each entry
- Resulting annotations match manually confirmed MS2 fragment peaks (e.g. for A1G1F(α1,3), 18 of 24 distinct peaks above 3% relative abundance were annotated)

## Limitations

- Fragment enumeration is capped at cutting one to seven bonds per glycan; fragments requiring more cuts are not generated
- The app has no de novo glycan identification capability — fragments can only be generated for glycans whose structure is explicitly provided
- The Fragment Generator supports the N-linked glycan scope of the app; O-linked glycans are not supported

## Evidence

- [abstract] The databases provided with the app can be expanded through the Fragment Generator functionality which automatically identifies fragmentation patterns for new glycans: "The databases provided with the app can be expanded through the Fragment Generator functionality which automatically identifies fragmentation patterns for new glycans"
- [discussion] The inclusion of a de novo approach could also enhance the app's ability to identify glycans not already located in the database.: "The inclusion of a de novo approach could also enhance the app’s ability to identify glycans not already located in the database."
- [readme] Welcome to GlyKAn AZ, an application developed to streamline the N-linked glycan identification process from LC-MS/MS data.: "Welcome to GlyKAn AZ, an application developed to streamline the N-linked glycan 
identification process from LC-MS/MS data."
