---
name: fragment-mass-table-generation
description: Use when a glycan of interest is absent from the supplied GlyKAn AZ MS1/MS2
  Fragment Database and theoretical fragment masses/patterns must be generated to
  expand the database before MS2 peak confirmation.
when_to_use_negative:
- The glycan is already present in the supplied MS1/MS2 Fragment Database, so fragment
  patterns are already available for MS2 Peak Confirmation
- The sample contains O-linked glycans — the app's scope is limited to N-linked glycans,
  so generated tables would not apply
- Only an MS spectrum is available with no known glycan structure string — the generator
  requires a defined structure and the app explicitly lacks de novo glycan identification
tools:
- name: GlyKAn AZ
  role: Hosts the Fragment Generator functionality that automatically identifies fragmentation
    patterns for new glycans to expand the app's databases
  repo: github.com/ZaylaSchaeffer/GlyKAn-AZ-application
- name: MATLAB
  role: 'Implementation environment: the glycan is represented as a MATLAB graph and
    fragments are derived via depth-first search within the app'
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1186/s12859-023-05346-5
    title: A MATLAB-based app to improve LC–MS/MS data analysis for N-linked glycan
      peak identification
  source_content_hash: sha256:3e584b6640a2a46eced00deba9257c5b8000a1e3508835cd4892f6a9c9122b9a
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
yes_rate: 1.0
n_claims: 8
---

# fragment-mass-table-generation

## Summary

Generates theoretical B-, Y-, and internal-ion fragment masses for an N-linked glycan not already in the GlyKAn AZ databases, by modeling the glycan as a MATLAB graph and cutting glycosidic bonds via depth-first search. The resulting fragment table expands the MS1/MS2 databases used for LC-MS/MS peak confirmation.

## When to use

Use when a glycan of interest is absent from the supplied GlyKAn AZ MS1/MS2 Fragment Database and theoretical fragment masses/patterns must be generated to expand the database before MS2 peak confirmation.

## When NOT to use

- The glycan is already present in the supplied MS1/MS2 Fragment Database, so fragment patterns are already available for MS2 Peak Confirmation
- The sample contains O-linked glycans — the app's scope is limited to N-linked glycans, so generated tables would not apply
- Only an MS spectrum is available with no known glycan structure string — the generator requires a defined structure and the app explicitly lacks de novo glycan identification

## Inputs

- Glycan structure string (monosaccharide residues and glycosidic connectivity) for a glycan not yet in the database
- Monosaccharide monoisotopic mass table (Xyl 150.0528, Fuc 164.0685, Hex 180.0634, HexN 221.0899, NAN 309.1060, etc.)
- MATLAB runtime environment with the GlyKAn AZ Fragment Generator

## Outputs

- fragment_table.csv listing each fragment's composition and theoretical monoisotopic mass
- Fragmentation pattern (B-, Y-, and internal ions from cutting 1-7 bonds) usable to expand the MS1/MS2 Fragment Database

## How to apply

Read the glycan structure string together with the monosaccharide mass table (Xyl 150.0528, Fuc 164.0685, Hex 180.0634, HexN 221.0899, NAN 309.1060 Da, etc.) in MATLAB. Parse the structure string into a graph whose nodes are monosaccharide residues and whose edges are glycosidic bonds, then perform a depth-first search that systematically cuts every combination of one to seven bonds, collecting the resulting connected subgraphs as B-, Y-, and internal-ion candidate fragments. Compute each fragment's theoretical monoisotopic mass by summing residue masses from the mass table with the appropriate terminal-group corrections, and write the fragment list (composition plus theoretical mass) to fragment_table.csv. Validate every emitted mass by checking that it decomposes exactly over the mass-table entries (e.g. 2 Hex = 360.1268, Hex + HexNAc = 401.1533); flag any unreproducible mass as a QC failure rather than admitting it into the fragment database.

## Related tools

- **GlyKAn AZ** (Hosts the Fragment Generator functionality that automatically identifies fragmentation patterns for new glycans to expand the app's databases) — github.com/ZaylaSchaeffer/GlyKAn-AZ-application
- **MATLAB** (Implementation environment: the glycan is represented as a MATLAB graph and fragments are derived via depth-first search within the app)

## Examples

```
assert abs(sum(MASS['Hex'] for _ in range(2)) - 360.1268) < 1e-4  # QC: fragment mass must decompose exactly over the monosaccharide mass table
```

## Evaluation signals

- Every emitted fragment mass decomposes exactly over the monosaccharide mass table entries; any mass that cannot be reproduced from the table is a QC failure
- Known-mass checks reproduce exactly: 2 Hex = 360.1268 Da and Hex + HexNAc = 401.1533 Da
- Output fragment set includes B-, Y-, and internal ions corresponding to cuts of one to seven bonds per glycan
- fragment_table.csv contains, for each fragment, both the composition and the theoretical monoisotopic mass
- Expanded database feeds the MS1/MS2 mass search so the new glycan can be confirmed at its observed retention time in subsequent runs

## Limitations

- Fragment generation enumerates cuts of only one to seven bonds per glycan; fragments requiring more cuts are not produced
- Requires an explicit glycan structure string — the app has no de novo glycan identification capability for glycans not already located in the database
- Scope is limited to N-linked glycans; O-linked glycans are not supported
- Repository README defers operational detail to the GlyKAn AZ Instructions Manual (Additional file 1), which is not included in the repo README itself

## Evidence

- [other] Fragment Generator graph representation: [evidence span withheld — anchor status: not_found]
- [abstract] Fragment Generator expands databases: "The databases provided with the app can be expanded through the Fragment Generator functionality which automatically identifies fragmentation patterns for new glycans"
- [other] Depth-first search bond cutting: [evidence span withheld — anchor status: not_found]
- [other] Fragment mass QC decomposition check: [evidence span withheld — anchor status: not_found]
- [readme] App purpose from README: "Welcome to GlyKAn AZ, an application developed to streamline the N-linked glycan identification process from LC-MS/MS data."
