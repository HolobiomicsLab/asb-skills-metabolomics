---
name: lipid-name-parsing-standardization
description: Use when lipidomics tables (csv/xlsx, flat or pivot, or mzTab-M) contain
  lipid names from different sources or dialects that must be mapped to standardized
  shorthand identifiers before computing structural distances, PCA projections, or
  per-sample quantity statistics.
when_to_use_negative:
- Names are already in a single standardized shorthand dialect and the analysis only
  needs exact string matching — parsing adds overhead without benefit
- Structures (SMILES/InChI or connection tables) are available directly, making name
  parsing unnecessary for graph-based structural comparison
- The molecules are not lipids covered by Goslin's supported categories (glycerophospholipids,
  glycerolipids, sphingolipids, etc.)
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3314
- http://edamontology.org/topic_0121
tools:
- name: cppgoslin
  role: C++ Goslin reference implementation whose two major functions — parsing and
    printing of lipid names — standardize lipid-name dialects into valid shorthand
    identifiers
  repo: https://github.com/lifs-tools/cppgoslin
- name: LipidSpace
  role: Standalone lipidomics analysis tool that embeds Goslin parsing at data import
    so every loaded lipidome row maps to a valid lipid identifier before structural
    and quantitative analyses
  repo: https://github.com/lifs-tools/lipidspace
provenance:
  source_task_ids:
  - task_008
  - task_004
  source_papers:
  - doi: 10.1021/acs.analchem.3c02449
    title: 'LipidSpace: Simple Exploration, Reanalysis, and Quality Control of Large-Scale
      Lipidomics Studies'
  source_content_hash: sha256:473c019fa5ad86b6d2d6e759abf0abd058929abd78cb289ab1582be3c51fd16d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lifs-tools/cppgoslin
  tools:
  - Goslin
  - LipidSpace
  - OpenBLAS
  - cppgoslin
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3314
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# lipid-name-parsing-standardization

## Summary

Parse and standardize heterogeneous lipid species names into valid lipid shorthand identifiers using the Goslin C++ library (cppgoslin), as done by LipidSpace prior to structural comparison, PCA modeling, and quality control. Unifying lipid-name dialects is a prerequisite for any downstream graph-based lipid distance computation.

## When to use

Use when lipidomics tables (csv/xlsx, flat or pivot, or mzTab-M) contain lipid names from different sources or dialects that must be mapped to standardized shorthand identifiers before computing structural distances, PCA projections, or per-sample quantity statistics.

## When NOT to use

- Names are already in a single standardized shorthand dialect and the analysis only needs exact string matching — parsing adds overhead without benefit
- Structures (SMILES/InChI or connection tables) are available directly, making name parsing unnecessary for graph-based structural comparison
- The molecules are not lipids covered by Goslin's supported categories (glycerophospholipids, glycerolipids, sphingolipids, etc.)

## Inputs

- Lipidome quantity table in csv or xlsx format (flat or pivot layout; lipid species row-wise or column-wise)
- Lipidomics data files in mzTab-M format
- Lipid species name strings from heterogeneous dialects (e.g., search-engine outputs from LipidXplorer, LDA, MS-DIAL)

## Outputs

- Per-row lipid shorthand identifiers parsed and standardized by Goslin, aligned with per-sample intensity/quantity columns
- A standardized lipid name table suitable for downstream n x n structural distance computation (LaWeCSE + Jaccard) and PCA projection

## How to apply

Load the lipidome quantity table (csv/xlsx flat or pivot layout, or mzTab-M) and pass each lipid species name through cppgoslin's parsing and printing functions so that every row maps to a valid lipid shorthand identifier. Goslin's grammars unify existing lipid-name dialects (e.g., the many glycerophospholipid class abbreviations such as BMP/LBPA or CDP-DAG), which LipidSpace relies on so that names can later be compared structurally via maximum common subgraph (LaWeCSE) plus Jaccard index rather than exact string matching. Preserve the parsed per-row identifiers alongside the intensity/quantity column per sample, since downstream steps (Benford's-law QC on leading digits, variance-normalized abundance addition to the PCA model) consume this standardized mapping. If parsing fails for a name, the row cannot participate in structural-space modeling; report it rather than silently dropping it. Note that cppgoslin is an early development version, so validate unusual names against the Goslin test files.

## Related tools

- **cppgoslin** (C++ Goslin reference implementation whose two major functions — parsing and printing of lipid names — standardize lipid-name dialects into valid shorthand identifiers) — https://github.com/lifs-tools/cppgoslin
- **LipidSpace** (Standalone lipidomics analysis tool that embeds Goslin parsing at data import so every loaded lipidome row maps to a valid lipid identifier before structural and quantitative analyses) — https://github.com/lifs-tools/lipidspace

## Examples

```
cd examples && make && ./lipid_name_parser
```

## Evaluation signals

- Every lipid name in the input table parses to a valid lipid shorthand identifier (no unresolved rows) before downstream distance computation
- Synonymous dialect abbreviations map to the same standardized class (e.g., BMP and LBPA to the same glycerophospholipid class)
- cppgoslin's own test suite passes: `make test` and `make runtests` complete without failures
- Downstream checks that depend on standardization succeed, e.g., the n x n distance matrix from LaWeCSE/Jaccard is computed for all distinct lipid species and the PCA model resolves the default seven principal components
- Names spanning several orders of magnitude in quantity still map correctly per sample for the Benford's-law QC leading-digit test

## Limitations

- cppgoslin is an early development version ('please use at your own risk'); unusual or non-standard lipid names may fail to parse
- Only lipid categories/classes covered by the Goslin grammars and test files are supported; lipid names outside these dialects cannot be standardized
- Names that cannot be parsed to a valid lipid shorthand cannot participate in structural-space (LaWeCSE/Jaccard) modeling, effectively excluding them from the lipid space model

## Evidence

- [methods] Goslin parses and standardizes lipid names at import: "It utilizes the C++ library of Goslin4,5 to parse and standardize lipid names"
- [readme] cppgoslin purpose statement: [evidence span withheld — anchor status: external_doc]
- [readme] cppgoslin's two major functions: [evidence span withheld — anchor status: external_doc]
- [readme] Import formats supported: "LipidSpace supports both csv and xlsx files for import. The tabels can be either pivot tables (that is lipid species in columns, samples in rows or lipid species in rows and samples in columns) or"
- [intro] Structural comparison relies on standardized names: "we applied the maximum common subgraph
(MCS) approach in combination with the Jaccard index to determine
a pairwise similarity of lipid species based on the molecular structure"
