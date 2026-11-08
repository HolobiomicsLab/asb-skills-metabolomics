---
name: metabolomics-annotation-counting
description: Use when comparing annotation coverage of competing open spectral libraries
  on the same MS2 feature table — e.g. asking how many extra annotations a new library
  adds over existing ones.
when_to_use_negative:
- When the input is already an annotated feature table and the goal is library-content
  curation rather than benchmarking annotation coverage.
- When MS2 spectra are absent for features — the workflow relies on MS2 spectra per
  feature for spectral matching.
- When libraries have inconsistent precursor mass or collision-energy metadata that
  would confound a direct count comparison.
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0153
tools:
- name: mzmine
  role: Processes raw MS data into feature tables and MS2 spectra used as FBMN input
  repo: https://github.com/mzmine/mzmine
- name: ThermoRawFileParser
  role: Converts Thermo RAW files to open formats (mzML/MGF) required by mzmine
  repo: https://github.com/CompOmics/ThermoRawFileParser
- name: MassBank-data
  role: Provides open MassBank records used as one of the reference spectral libraries
    in the benchmark
  repo: https://github.com/MassBank/MassBank-data
- name: msn_tree_library
  role: Repository supplying MSnLib reference entries and metadata templates for library
    construction
  repo: https://github.com/corinnabrungs/msn_tree_library
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41592-025-02813-0
    title: 'MSnLib: efficient generation of open multi-stage fragmentation mass spectral
      libraries'
  source_content_hash: sha256:57018366a31ee48caacab41f09a5898d40037ad697a4af454dfdba2df2547c66
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mzmine/mzmine
  tools:
  - MassBank-data
  - MassBank/MassBank-data
  - ThermoRawFileParser
  - msn_tree_library
  - mzmine
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0153
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# metabolomics-annotation-counting

## Summary

Quantify how many MS2 annotations each spectral library (GNPS, MoNA, MassBank EU, MSnLib) yields on a shared feature-based molecular networking run of a public dataset, then compute per-library and unique-match counts for direct benchmarking.

## When to use

Use when comparing annotation coverage of competing open spectral libraries on the same MS2 feature table — e.g. asking how many extra annotations a new library adds over existing ones.

## When NOT to use

- When the input is already an annotated feature table and the goal is library-content curation rather than benchmarking annotation coverage.
- When MS2 spectra are absent for features — the workflow relies on MS2 spectra per feature for spectral matching.
- When libraries have inconsistent precursor mass or collision-energy metadata that would confound a direct count comparison.

## Inputs

- MS2 raw files from a public MassIVE dataset (e.g. group1_B* files of MSV000096589)
- mzmine feature table and associated MS2 spectra
- Spectral library files: GNPS, MoNA, MassBank EU records, and MSnLib reference entries

## Outputs

- Per-library annotation counts (e.g. 129 open-library vs 80 MSnLib)
- Total unique annotation count across all libraries (e.g. 150)
- Number of annotations unique to one library (e.g. 21 unique MSnLib matches)
- FBMN annotation output listing matched compound per feature

## How to apply

Retrieve the MS2 raw files of the target MassIVE dataset (e.g. the group1_B* files from MSV000096589, drug-incubated bacterial cultures). Convert Thermo RAW with ThermoRawFileParser, then process in mzmine to produce a feature table plus MS2 spectra for Feature-Based Molecular Networking. Run FBMN with spectral library searches against GNPS, MoNA and MassBank EU in one arm, and against MSnLib reference entries in a parallel arm, using identical feature tables so only the library differs. Extract per-library annotation counts from the annotation output and compute the union and unique matches: for MSV000096589 this gave 129 open-library, 80 MSnLib, 150 total and 21 unique MSnLib annotations. Validate by confirming each count matches the reported values before drawing any coverage conclusions.

## Related tools

- **mzmine** (Processes raw MS data into feature tables and MS2 spectra used as FBMN input) — https://github.com/mzmine/mzmine
- **ThermoRawFileParser** (Converts Thermo RAW files to open formats (mzML/MGF) required by mzmine) — https://github.com/CompOmics/ThermoRawFileParser
- **MassBank-data** (Provides open MassBank records used as one of the reference spectral libraries in the benchmark) — https://github.com/MassBank/MassBank-data
- **msn_tree_library** (Repository supplying MSnLib reference entries and metadata templates for library construction) — https://github.com/corinnabrungs/msn_tree_library

## Examples

```
python feature_based_molecular_networking.py --input feature_table.csv --ms2 spectra.mgf --library msnlib/ --output annotations.csv
```

## Evaluation signals

- Per-library annotation counts must match published values (129 open-library, 80 MSnLib on MSV000096589).
- Total unique annotations across libraries must equal 150 on this dataset.
- Unique MSnLib matches must equal 21, of which 14 correspond to added drugs.
- Feature table and MS2 spectra must be identical across library arms — only the search library may differ.

## Limitations

- Counts depend on the mzmine processing parameters and the FBMN search settings; changing them shifts absolute numbers.
- Only MS2 data are used; MSnLib's multi-stage (MSn tree) content beyond MS2 is not exercised by this counting workflow.
- The comparison assumes all libraries are queried with the same tolerances; unadjusted differences reflect library content, not method sensitivity.

## Evidence

- [methods] workflow: [evidence span withheld — anchor status: not_found]
- [results] finding: [evidence span withheld — anchor status: not_found]
- [readme] readme: [evidence span withheld — anchor status: external_doc]
