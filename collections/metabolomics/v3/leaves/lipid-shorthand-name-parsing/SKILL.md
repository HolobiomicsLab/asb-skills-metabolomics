---
name: lipid-shorthand-name-parsing
description: Use when lipid identifiers arrive as heterogeneous shorthand names (LIPID
  MAPS, SwissLipids, or search-engine dialects from LipidXplorer, LDA, MS-DIAL) and
  must be converted to a canonical, machine-comparable structure before distance computation
  or structural comparison.
when_to_use_negative:
- Input is already a canonical structural representation (e.g., SMILES or atom/bond
  graphs) rather than a shorthand lipid name
- The lipid names are free-text descriptions or non-standard abbreviations that Goslin's
  grammars cannot tokenize — the parser will fail rather than produce a reliable structure
- The analysis only requires exact name matching between lipidomes and no structural
  comparison, in which case parsing to structures is unnecessary overhead
tools:
- name: cppgoslin (Goslin)
  role: Reference C++ implementation of the Goslin grammar used to parse and standardize
    lipid names into structured components
  repo: https://github.com/lifs-tools/cppgoslin
- name: LipidSpace
  role: Bundles libcppGoslin and invokes the parser during data import to standardize
    lipid names before structural comparison
  repo: https://github.com/lifs-tools/lipidspace
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/acs.analchem.3c02449
    title: 'LipidSpace: Simple Exploration, Reanalysis, and Quality Control of Large-Scale
      Lipidomics Studies'
  source_content_hash: sha256:1fd59ad3704932293fda04e315388a9ef362480350251c78c0f8b40e08ebd41c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lifs-tools/cppgoslin
  tools:
  - Goslin
  - LIPID MAPS
  - LipidSpace
  - cppgoslin (Goslin)
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# lipid-shorthand-name-parsing

## Summary

Parses and standardizes lipid shorthand names (e.g., FA 18:2(9Z,12Z), PC 18:0_16:1) into structured components — headgroup, fatty acyl chains, DB positions and sn-positions — using the Goslin grammar implementation (cppgoslin), which LipidSpace embeds as the first step of every analysis.

## When to use

Use when lipid identifiers arrive as heterogeneous shorthand names (LIPID MAPS, SwissLipids, or search-engine dialects from LipidXplorer, LDA, MS-DIAL) and must be converted to a canonical, machine-comparable structure before distance computation or structural comparison.

## When NOT to use

- Input is already a canonical structural representation (e.g., SMILES or atom/bond graphs) rather than a shorthand lipid name
- The lipid names are free-text descriptions or non-standard abbreviations that Goslin's grammars cannot tokenize — the parser will fail rather than produce a reliable structure
- The analysis only requires exact name matching between lipidomes and no structural comparison, in which case parsing to structures is unnecessary overhead

## Inputs

- Lipid shorthand name strings from heterogeneous dialects (e.g., 'FA 18:2(9Z,12Z)', 'PC 18:0_16:1')
- Lipidomics import tables in csv/xlsx (flat or pivot layout) or mzTab-M containing a lipid-name column
- Lipid structure records from databases such as LIPID MAPS and SwissLipids

## Outputs

- Standardized/normalized lipid names parsed into structured components (headgroup, fatty acyl chains, DB positions, sn-positions)
- Parsed records feeding downstream structural graph construction and MCS-based distance computation

## How to apply

Pass each lipid name string through the Goslin parser (cppgoslin, bundled in LipidSpace as libcppGoslin) to decompose it into a normalized representation: headgroup class, fatty acyl chain lengths/unsaturations, double-bond positions with stereochemistry such as 9Z,12Z, and sn-position annotations. The parser unifies all existing lipid-name dialects, so mixed-source tables (flat or pivot csv/xlsx, or mzTab-M) can be standardized at import time before any downstream processing. Preserve whether the sn-position is specified (e.g., PC 18:0_16:1 vs PC 18:0/16:1), because LipidSpace uses this flag to decide whether fatty-acyl chains are compared positionally or via all combinations ('Ignore lipid sn-positions' mode). Names that fail to parse should be rejected or flagged rather than silently passed on, since the MCS/Jaccard structural comparison depends on correct atom-level reconstruction from the parsed components.

## Related tools

- **cppgoslin (Goslin)** (Reference C++ implementation of the Goslin grammar used to parse and standardize lipid names into structured components) — https://github.com/lifs-tools/cppgoslin
- **LipidSpace** (Bundles libcppGoslin and invokes the parser during data import to standardize lipid names before structural comparison) — https://github.com/lifs-tools/lipidspace

## Examples

```
cd examples && make && ./lipid_name_parser  # build and run the cppgoslin parsing example
```

## Evaluation signals

- Parsed names round-trip to canonical shorthand with correct DB positions and stereochemistry (e.g., linoleic acid parses to FA 18:2(9Z,12Z), oleic acid to FA 18:1(9Z))
- cppgoslin's own test suite passes: `make test` and `make runtests` complete without failures
- sn-position information is preserved: names like 'PC 18:0_16:1' remain distinguishable from sn-specified '/' notation, enabling LipidSpace's all-combinations comparison mode
- Unparseable or ambiguous names are flagged/errored rather than silently producing a wrong structure that corrupts downstream MCS/Jaccard distances
- All lipid names in an imported table are standardized to a single dialect before distance matrices are computed

## Limitations

- cppgoslin is explicitly an early development version ('please use at your own risk'), so parse failures or grammar gaps should be reported to the developers
- Dialect coverage is limited to the lipid classes defined in the Goslin grammars; unsupported class abbreviations will not parse
- Names lacking sn-position specification force combinatorial chain comparison in downstream analysis, decreasing performance (more accurate results but slower)

## Evidence

- [methods] Goslin parses names during import: "It utilizes the C++ library of Goslin4,5 to parse and standardize lipid names"
- [methods] Import formats and layouts supported: "Several table formats are supported for the import, such as flat or pivot tables where lipid content is stored row-wise and the sample content column-wise or vice versa."
- [readme] cppgoslin is the Goslin reference implementation for C++: [evidence span withheld — anchor status: external_doc]
- [readme] Goslin unifies lipid-name dialects: [evidence span withheld — anchor status: external_doc]
- [readme] sn-position ambiguity affects chain comparison mode: "when the sn-position is not specified as for instance in PC 18:0_16:1, a mode can be activated to compare all combinations of FA comparisons"
