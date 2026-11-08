---
name: lipid-nomenclature-parsing
description: Use when lipidomics data tables contain lipid species names in mixed
  dialects (e.g., LIPID MAPS, SwissLipids, shorthand like PC 18:0_16:1) that must
  be parsed and standardized before structural comparison, PCA, or feature analysis.
when_to_use_negative:
- The input is already a parsed/standardized feature matrix (e.g., numerical abundances
  keyed by canonical identifiers) with no lipid name strings left to parse
- The lipid identifiers fall outside the dialects and lipid classes covered by the
  Goslin grammars (e.g., non-standard shorthand the parser cannot unify)
edam_operation: http://edamontology.org/operation_3282
edam_topics:
- http://edamontology.org/topic_0153
- http://edamontology.org/topic_3172
tools:
- name: cppgoslin
  role: C++ Goslin library used by LipidSpace to parse and standardize lipid names
    at import time
  repo: https://github.com/lifs-tools/cppgoslin
- name: LipidSpace
  role: Workstation application that invokes Goslin during data import and uses the
    parsed structures for MCS/Jaccard-based structural comparison
  repo: https://github.com/lifs-tools/lipidspace
provenance:
  source_task_ids:
  - task_003
  - task_006
  source_papers:
  - doi: 10.1021/acs.analchem.3c02449
    title: 'LipidSpace: Simple Exploration, Reanalysis, and Quality Control of Large-Scale
      Lipidomics Studies'
  source_content_hash: sha256:3126783b55c2ecbf00be934dc85d160156bd4912dc8e93dbd5185530acf70bb4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lifs-tools/cppgoslin
  tools:
  - Goslin
  - LIPID MAPS
  - LipidSpace
  - OpenBLAS
  - cppgoslin
  edam_operation: http://edamontology.org/operation_3282
  edam_topics:
  - http://edamontology.org/topic_0153
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# lipid-nomenclature-parsing

## Summary

Standardizes heterogeneous lipid species names into a common structural representation by parsing them with Goslin (cppgoslin), the prerequisite step that lets LipidSpace compare lipidomes, compute structural distances, and evaluate double bond annotation correctness.

## When to use

Use when lipidomics data tables contain lipid species names in mixed dialects (e.g., LIPID MAPS, SwissLipids, shorthand like PC 18:0_16:1) that must be parsed and standardized before structural comparison, PCA, or feature analysis.

## When NOT to use

- The input is already a parsed/standardized feature matrix (e.g., numerical abundances keyed by canonical identifiers) with no lipid name strings left to parse
- The lipid identifiers fall outside the dialects and lipid classes covered by the Goslin grammars (e.g., non-standard shorthand the parser cannot unify)

## Inputs

- Lipidomics data tables in csv or xlsx format, as flat or pivot tables (lipid species row-wise or column-wise)
- Lipidomics data files in mzTab-M format
- Individual lipid species names in heterogeneous dialects (e.g., PC 18:0_16:1, LIPID MAPS or SwissLipids identifiers)

## Outputs

- Standardized lipid species representations with parsed headgroup, fatty acyl chain composition, sn-positions, and double bond positions
- Imported, quality-controlled LipidSpace analysis ready for structural space computation and feature analysis

## How to apply

Run each lipid species name through the C++ Goslin library (cppgoslin, bundled in LipidSpace as libcppGoslin), which unifies all existing lipid name dialects into a standardized representation including headgroup, fatty acyl chains, and double bond positions. LipidSpace applies this parsing at data import: csv/xlsx tables in flat or pivot layout, or mzTab-M files, are read and every lipid identifier is parsed and standardized before any downstream computation. The parsed sn-position information drives a decision point: by default the first FA of one lipid is compared to the first FA of the other, but when the sn-position is unspecified (e.g., PC 18:0_16:1) a mode can be activated in the menu under Analysis → Ignore lipid sn-positions to compare all FA combinations and pick the lowest distance. Parsing correctness can be validated against the Goslin grammars and test files maintained in the goslin project, and downstream against reference databases (LIPID MAPS, SwissLipids) extracted at the highest structural resolution level.

## Related tools

- **cppgoslin** (C++ Goslin library used by LipidSpace to parse and standardize lipid names at import time) — https://github.com/lifs-tools/cppgoslin
- **LipidSpace** (Workstation application that invokes Goslin during data import and uses the parsed structures for MCS/Jaccard-based structural comparison) — https://github.com/lifs-tools/lipidspace

## Examples

```
cd examples && make && ./lipid_name_parser
```

## Evaluation signals

- Every imported lipid name resolves without parse errors; cppgoslin 'make test' / 'make runtests' pass on the bundled grammar test files
- Parsed headgroups and fatty acyl compositions match the expected classes in the cppgoslin supported-lipids table (e.g., GP classes such as PC, BMP/LBPA, CDP-DAG)
- Downstream invariant: the 14 PC 12:0/[12-24,26]:0 lipids retain their sequential arc-like positioning in structural space after name parsing (Supporting Figure S7)
- Double bond positions recovered from parsed names reproduce the SwissLipids mismatch probabilities (~64% forward, >71% backward counting from the methyl end)
- Lipid count in the imported analysis matches the row/column count of the source csv/xlsx or mzTab-M file

## Limitations

- When the sn-position is not specified (e.g., PC 18:0_16:1), default first-to-first FA comparison can be inaccurate; the all-combinations mode is more accurate but decreases performance
- The distance metric choice trades accuracy against interpretability: the unbound distance mode gives more accurate results but reduces visibility of lipid spaces because distances may become very large
- cppgoslin is an early development version; its own README advises using it at your own risk and reporting issues

## Evidence

- [methods] Goslin parses and standardizes lipid names: "It utilizes the C++ library of Goslin4,5 to parse and standardize lipid names"
- [methods] Import formats supported: "Several table formats are supported for the import, such as flat or pivot tables where lipid content is stored row-wise and the sample content column-wise or vice versa."
- [readme] sn-position comparison decision point: "However, when the sn-position is not specified as for instance in PC 18:0_16:1, a mode can be activated to compare all combinations of FA comparisons for both lipids"
- [readme] Goslin purpose: [evidence span withheld — anchor status: external_doc]
