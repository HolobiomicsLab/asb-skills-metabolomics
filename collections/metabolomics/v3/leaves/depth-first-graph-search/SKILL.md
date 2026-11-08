---
name: depth-first-graph-search
description: Use when theoretical B/Y/internal fragment masses must be enumerated
  for an N-linked glycan structure absent from the MS1/MS2 databases, by systematically
  cutting one to seven glycosidic bonds in a graph representation of the structure.
when_to_use_negative:
- The goal is de novo glycan identification from MS/MS spectra — the app itself is
  limited to glycans in its database and does not perform de novo identification
- The input structures are O-linked glycans — the app's scope is limited to N-linked
  glycans
- The task is peak identification on an existing mzXML file rather than generating
  new database entries — the MS1/MS2 search tabs handle that directly
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: GlyKAn AZ
  role: Hosts the Fragment Generator in which the depth-first bond-cutting search
    enumerates fragments for glycans not already in the database
  repo: github.com/ZaylaSchaeffer/GlyKAn-AZ-application
- name: MATLAB
  role: 'Implementation language: the glycan structure string is parsed into a MATLAB
    graph and the depth-first search and mass computations run in MATLAB'
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1186/s12859-023-05346-5
    title: A MATLAB-based app to improve LC–MS/MS data analysis for N-linked glycan
      peak identification
  source_content_hash: sha256:6bc56e72cc7e94751e2bff4d999582c0db6526b6cdfc13f289c08074dfe897a8
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
n_claims: 10
---

# depth-first-graph-search

## Summary

Exhaustive depth-first traversal of a glycan represented as a MATLAB graph (nodes = monosaccharides, edges = glycosidic bonds) that cuts every combination of 1–7 bonds to enumerate B-, Y-, and internal-ion fragments and their theoretical monoisotopic masses, as implemented in the GlyKAn AZ Fragment Generator.

## When to use

Use when theoretical B/Y/internal fragment masses must be enumerated for an N-linked glycan structure absent from the MS1/MS2 databases, by systematically cutting one to seven glycosidic bonds in a graph representation of the structure.

## When NOT to use

- The goal is de novo glycan identification from MS/MS spectra — the app itself is limited to glycans in its database and does not perform de novo identification
- The input structures are O-linked glycans — the app's scope is limited to N-linked glycans
- The task is peak identification on an existing mzXML file rather than generating new database entries — the MS1/MS2 search tabs handle that directly

## Inputs

- Glycan structure string (residues and glycosidic linkages) for a glycan not already in the MS2 Fragment Database
- Monosaccharide monoisotopic mass table (e.g. Xyl 150.0528, Fuc 164.0685, Hex 180.0634, HexN 221.0899, NAN 309.1060)
- MATLAB environment running the GlyKAn AZ app

## Outputs

- fragment_table.csv listing fragment composition and theoretical monoisotopic mass for each generated B/Y/internal ion
- Updated MS2 Fragment Database entries (fragmentation patterns) usable by the MS2 Peak Confirmation tab
- QC flags for any fragment mass that fails to decompose over the monosaccharide mass table

## How to apply

Read the glycan structure string together with a monosaccharide mass table (Xyl 150.0528, Fuc 164.0685, Hex 180.0634, HexN 221.0899, NAN 309.1060, etc.) in MATLAB, and parse the structure string into a graph whose nodes are monosaccharide residues and whose edges are glycosidic bonds. Perform a depth-first search over the graph that systematically cuts every combination of one to seven bonds, collecting the resulting connected subgraphs as candidate fragments (B-, Y-, and internal ions). Compute each fragment's theoretical monoisotopic mass by summing residue masses from the table with appropriate terminal-group corrections. Write the fragment list (composition and theoretical mass) to fragment_table.csv. Validate by checking that every emitted fragment mass decomposes exactly over the monosaccharide mass table entries — e.g. 2 Hex must equal 360.1268 and Hex + HexNAc must equal 401.1533 — and flag any mass that cannot be reproduced from the table as a QC failure. This expands the GlyKAn AZ MS2 Fragment Database so new glycans become searchable in subsequent MS2 Peak Confirmation runs.

## Related tools

- **GlyKAn AZ** (Hosts the Fragment Generator in which the depth-first bond-cutting search enumerates fragments for glycans not already in the database) — github.com/ZaylaSchaeffer/GlyKAn-AZ-application
- **MATLAB** (Implementation language: the glycan structure string is parsed into a MATLAB graph and the depth-first search and mass computations run in MATLAB)

## Evaluation signals

- Every emitted fragment mass decomposes exactly over the monosaccharide mass table entries; any non-decomposable mass is a QC failure
- Spot-check arithmetic: 2 Hex fragments must equal 360.1268 Da and Hex + HexNAc fragments must equal 401.1533 Da
- Fragments correspond to connected subgraphs obtained by cutting 1–7 bonds of the input glycan graph — no disconnected or >7-cut fragments should appear
- fragment_table.csv is produced and its entries are loadable as MS2 Fragment Database records for subsequent MS2 Peak Confirmation runs

## Limitations

- Fragments are generated only by cutting one to seven bonds per glycan; larger multi-cut fragmentation is not enumerated
- The generator expands the database for glycans provided as structure strings; it does not add de novo identification capability to the app
- Fragment generation quality depends on the monosaccharide mass table; masses outside the table cannot be validated

## Evidence

- [methods] Fragment Generator depth-first enumeration: [evidence span withheld — anchor status: not_found]
- [abstract] Database expansion via Fragment Generator: "The databases provided with the app can be expanded through the Fragment Generator functionality which automatically identifies fragmentation patterns for new glycans"
- [readme] Repository provenance and license: "This app was developed by Zayla Schaeffer and Ashna Dhingra at AstraZeneca, and is available under MIT license (see License.txt)."
