---
name: lipidomics-data-table-import-flat-pivot-csv-xlsx
description: Use when the input is a lipidomics abundance table (csv, xlsx, or mzTab-M)
  with lipid species names in row-wise or column-wise (pivot) orientation that must
  be loaded and its heterogeneous lipid nomenclature standardized.
when_to_use_negative:
- Input is already a standardized feature/abundance matrix keyed by valid lipid shorthand
  identifiers — re-parsing adds no value
- Input is raw mass-spectrometric peak data from a lipid search engine run; this skill
  expects the search engine's quantitative output table (e.g. from LipidXplorer, LDA,
  or MS-DIAL), not spectra
- The goal is structural similarity computation itself — import is only the prerequisite
  step, not the analysis
tools:
- name: LipidSpace
  role: Workstation application that performs the import of flat/pivot csv and xlsx
    tables and mzTab-M files into lipidome objects for analysis
  repo: https://github.com/lifs-tools/lipidspace
- name: Goslin (cppgoslin)
  role: C++ library bundled with LipidSpace to parse and standardize lipid names from
    all existing naming dialects during import
  repo: https://github.com/lifs-tools/cppgoslin
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/acs.analchem.3c02449
    title: 'LipidSpace: Simple Exploration, Reanalysis, and Quality Control of Large-Scale
      Lipidomics Studies'
  source_content_hash: sha256:4dd036a4801bd61dd5186c74346d4ec8121d4bd859511b91d5f700496e46c8e1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lifs-tools/lipidspace
  tools:
  - Goslin
  - Goslin (cppgoslin)
  - LipidSpace
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# lipidomics-data-table-import-flat-pivot-csv-xlsx

## Summary

Import lipidomics quantity tables (flat or pivot layout, csv/xlsx, or mzTab-M) into an analysis environment, parsing and standardizing lipid species names with the Goslin library so each row/column maps to a valid lipid shorthand identifier before downstream lipidome comparison or QC.

## When to use

Use when the input is a lipidomics abundance table (csv, xlsx, or mzTab-M) with lipid species names in row-wise or column-wise (pivot) orientation that must be loaded and its heterogeneous lipid nomenclature standardized.

## When NOT to use

- Input is already a standardized feature/abundance matrix keyed by valid lipid shorthand identifiers — re-parsing adds no value
- Input is raw mass-spectrometric peak data from a lipid search engine run; this skill expects the search engine's quantitative output table (e.g. from LipidXplorer, LDA, or MS-DIAL), not spectra
- The goal is structural similarity computation itself — import is only the prerequisite step, not the analysis

## Inputs

- Lipidomics quantity table in csv or xlsx format, flat or pivot layout (lipid species in rows with samples in columns, or vice versa)
- Lipidomics data file in mzTab-M format
- Sample-associated study variable metadata (e.g. body mass index, tissue, condition)

## Outputs

- Tabular lipidome data frame with standardized lipid shorthand identifiers and per-sample quantity columns
- Lipidome objects (one per sample) ready for structural comparison, feature analysis, and QC

## How to apply

Load the table into a tabular data frame, detecting whether lipid content is stored row-wise and sample content column-wise or vice versa (LipidSpace supports both orientations plus flat tables). Parse every lipid name with the Goslin library (e.g. cppgoslin in LipidSpace) so that all existing dialects of lipid names are unified into valid lipid shorthand identifiers; rows failing to parse indicate malformed species names that must be corrected or dropped. Extract the intensity/quantity column per sample alongside any sample-associated study variables (e.g. body mass index, tissue, condition) for combined exploration. A bundled tutorial/example dataset (e.g. the LipidSpace example dataset via File → Import example dataset, or examples/Example-Dataset.xlsx) can be used as a reference input. Rationale: exact string matches across heterogeneous lipidomes miss structural relationships, so standardized Goslin identifiers are the prerequisite for later maximum-common-subgraph/Jaccard distance computation and Benford's-law QC of quantity values.

## Related tools

- **LipidSpace** (Workstation application that performs the import of flat/pivot csv and xlsx tables and mzTab-M files into lipidome objects for analysis) — https://github.com/lifs-tools/lipidspace
- **Goslin (cppgoslin)** (C++ library bundled with LipidSpace to parse and standardize lipid names from all existing naming dialects during import) — https://github.com/lifs-tools/cppgoslin

## Examples

```
cd examples && make && ./lipid_name_parser  # cppgoslin demo: parse lipid names to standardized shorthand as done during LipidSpace import
```

## Evaluation signals

- Every lipid species name resolves through Goslin to a valid lipid shorthand identifier; unparseable names are surfaced rather than silently kept
- Both orientations of pivot tables (lipids-in-rows/samples-in-columns and the transpose) yield the same parsed lipidome set
- Imported quantity values per sample span the expected dynamic range (several orders of magnitude) so downstream Benford's-law QC can pass
- The bundled example dataset (File → Import example dataset) imports without errors and appears in the sample selection tab
- mzTab-M input produces the same lipidome structure as the equivalent csv/xlsx table

## Limitations

- LipidSpace analyses cannot yet be stored or reloaded — the README states 'At this moment it is not possible to store or load an analysis', so imported sessions are ephemeral
- Lipid name standardization depends on Goslin grammar coverage; names outside supported lipid categories/classes will fail to parse
- The tool is designed as a workstation application for a regular office computer, constraining the scale of tables handled per run

## Evidence

- [methods] Several table formats are supported for the import, such as flat or pivot tables where lipid content is stored row-wise and the sample content column-wise or vice versa.: "Several table formats are supported for the import, such as flat or pivot tables where lipid content is stored row-wise and the sample content column-wise or vice versa."
- [methods] It utilizes the C++ library of Goslin to parse and standardize lipid names: "It utilizes the C++ library of Goslin4,5 to parse and standardize lipid names"
- [readme] LipidSpace supports both csv and xlsx files for import. The tabels can be either pivot tables (that is lipid species in columns, samples in rows or lipid species in rows and samples in columns) or flat tables.: "LipidSpace supports both csv and xlsx files for import. The tabels can be either pivot tables (that is lipid species in columns, samples in rows or lipid species in rows and samples in columns) or"
- [readme] You can open our provided example dataset in the menu → File → Import example dataset.: "You can open our provided example dataset in the menu → File → Import example dataset."
- [intro] Sample-associated study variables (such as body mass index, tissue, and condition) can be added to an analysis for combined exploration: "Sample-associated study variables (such as body mass index, tissue,
and condition) can be added to an analysis for combined exploration"
