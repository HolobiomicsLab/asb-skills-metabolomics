---
name: mwtab-format-parsing
description: Use when input files are Metabolomics Workbench mwTab-format or JSONized
  analysis files (or ANxxxxxx analysis IDs) that must be tokenized into MWTabFile
  objects, converted between formats, or schema-validated.
when_to_use_negative:
- Input is raw instrument data (mzML, NetCDF, NMR FIDs) not already deposited in mwTab
  format — mwtab only reads/writes mwTab and its JSONized form
- The payload is a non-mwTab REST response (csv/json/text) not representable in mwTab
  format — hold it as a raw MWRESTFile instance instead of forcing MWTabFile parsing
- The goal is statistical analysis of metabolite abundances rather than format parsing,
  validation, or metadata-heading matching
edam_operation: http://edamontology.org/operation_0226
edam_topics:
- http://edamontology.org/topic_3366
- http://edamontology.org/topic_0091
tools:
- name: mwtab
  role: Core library providing MWTabFile, fileio.read_files, tokenizer, and mwTab<->JSON
    conversion for parsing and validating Metabolomics Workbench files
  repo: https://github.com/MoseleyBioinformaticsLab/mwtab
- name: Python
  role: Runtime (3.6+) in which the mwtab package runs as a library or command-line
    tool
- name: jsonschema
  role: Backs the validate()/validate_file() schema checking of parsed mwTab data
- name: pandas
  role: Handles tabular data sections (e.g., METABOLITES columns) extracted from parsed
    mwTab files
provenance:
  source_task_ids:
  - task_002
  - task_004
  - task_001
  - task_006
  source_papers:
  - doi: 10.3390/metabo11030163
    title: The mwtab Python Library for RESTful Access and Enhanced Quality Control,
      Deposition, and Curation of the Metabolomics Workbench Data Repository
  source_content_hash: sha256:03778eb38dbc00744a5951caf3e50dcd2c83883406336bf9e2bcd0475256992f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/MoseleyBioinformaticsLab/mwtab
  tools:
  - Python
  - jsonschema
  - mwtab
  - pandas
  edam_operation: http://edamontology.org/operation_0226
  edam_topics:
  - http://edamontology.org/topic_3366
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 16
---

# mwtab-format-parsing

## Summary

Parse, validate, and interconvert Metabolomics Workbench 'mwTab' text files and their JSONized equivalents using the mwtab Python library (MWTabFile, mwtab.fileio.read_files, mwtab.tokenizer), enabling downstream QC and metadata matching on MS/NMR metabolomics datasets.

## When to use

Use when input files are Metabolomics Workbench mwTab-format or JSONized analysis files (or ANxxxxxx analysis IDs) that must be tokenized into MWTabFile objects, converted between formats, or schema-validated.

## When NOT to use

- Input is raw instrument data (mzML, NetCDF, NMR FIDs) not already deposited in mwTab format — mwtab only reads/writes mwTab and its JSONized form
- The payload is a non-mwTab REST response (csv/json/text) not representable in mwTab format — hold it as a raw MWRESTFile instance instead of forcing MWTabFile parsing
- The goal is statistical analysis of metabolite abundances rather than format parsing, validation, or metadata-heading matching

## Inputs

- mwTab-format text files from the Metabolomics Workbench (e.g., for analyses AN000001–AN001891)
- JSONized equivalents of mwTab files
- Metabolomics Workbench analysis IDs or study/analysis ID pairs fetched over the REST interface
- standard_column_names.json with standardized metabolite metadata column names

## Outputs

- MWTabFile objects whose keys are section names (e.g., METABOLITES, SUBJECT_SAMPLE_FACTORS, _DATA)
- JSONized representations of mwTab files and vice versa
- QC tallies such as download/parse failure counts (e.g., 3 mwTab and 50 JSON un-downloadable, 70 mwTab and 139 JSON un-parseable)
- Per-column match counts and per-file flags for standardized metabolite metadata fields (e.g., 1216 of 1818 files with matching common fields)

## How to apply

Install mwtab 1.0.1 under Python 3.6+ (python3 -m pip install mwtab) and read files either from local paths or directly from the MW REST interface via mwtab.mwrest, letting read_files return MWTabFile instances keyed by section (e.g., METABOLITES, SUBJECT_SAMPLE_FACTORS, _DATA). Use the mwtab.tokenizer machinery for line-by-line tokenization, since section keys, item keys/values, and tabular sections must be preserved exactly when comparing mwTab text against its JSON representation. To convert, use the package's bidirectional mwTab<->JSON conversion so a JSONized file can be written back and compared for consistency (as in the repository-wide check where 1345 of ~1685 paired analyses mismatched between formats). Validate parsed files against the MS/NMR required schemas via the validate()/validate_file() functions backed by jsonschema before treating records as curatable. For tabular sections, hand the parsed blocks to pandas, and when matching metabolite metadata headings (pubchem_id, inchi_key, kegg_id, hmdb_id, retention_index, retention_time), use the ColumnFinder/NameMatcher/ValueMatcher dict_match and series_match methods built from regular expressions assembled with make_list_regex.

## Related tools

- **mwtab** (Core library providing MWTabFile, fileio.read_files, tokenizer, and mwTab<->JSON conversion for parsing and validating Metabolomics Workbench files) — https://github.com/MoseleyBioinformaticsLab/mwtab
- **Python** (Runtime (3.6+) in which the mwtab package runs as a library or command-line tool)
- **jsonschema** (Backs the validate()/validate_file() schema checking of parsed mwTab data)
- **pandas** (Handles tabular data sections (e.g., METABOLITES columns) extracted from parsed mwTab files)

## Examples

```
import mwtab
for mwfile in mwtab.read_files("1", "2"):
    print("STUDY_ID:", mwfile.study_id); print("Blocks:", list(mwfile.keys()))
```

## Evaluation signals

- MWTabFile sections enumerate expected keys, and STUDY_ID/ANALYSIS_ID/SOURCE round-trip correctly through read_files
- A mwTab file converted to JSON and back reproduces the original section keys, item keys/values, SUBJECT_SAMPLE_FACTORS, and _DATA content with zero mismatches
- Validation via mwschema.ms_required_schema / nmr_required_schema passes without reported errors for well-formed depositions
- File-level tallies reproduce reported QC figures, e.g., 1818 downloadable mwTab files, 1216 with standardized metabolite metadata headings, 1345/1685 mwTab-vs-JSON inconsistencies
- Named problem analyses (AN001492, AN001493, AN001499, AN002428) show the expected defects, e.g., JSON files missing a 'Data' item

## Limitations

- Some analyses return blank pages over the REST interface (3 in mwTab format, 50 in JSON format as of the 19 November 2020 snapshot) and cannot be retrieved
- Many downloaded files fail parsing (70 mwTab and 139 JSON un-parseable), and cross-format inconsistency is common (1345 analyses), so the mwTab and JSONized representations cannot be assumed interchangeable
- Non-mwTab REST responses (csv/json/text) cannot be parsed with the standard MWTabFile machinery and must be retained as raw MWRESTFile content
- The package runs under Python 3.6+ only

## Evidence

- [readme] The ``mwtab`` package is a Python library that facilitates reading and writing files in ``mwTab`` format used by the `Metabolomics Workbench`_ for archival of Mass Spectrometry (MS) and Nuclear Magnetic Resonance (NMR) experimental data.: "The ``mwtab`` package is a Python library that facilitates reading and writing files in ``mwTab`` format used by the `Metabolomics Workbench`_"
- [methods] The :mod:`mwtab` package runs under Python 3.6+.: "The :mod:`mwtab` package runs under Python 3.6+."
- [methods] Validate data stored in ``mwTab`` file based on schema definition.: "Validate data stored in ``mwTab`` file based on schema definition."
- [methods] code was created to be able to identify the columns under their various names as well as evaluate the format of their values: "code was created to be able to identify the columns under their various names as well as evaluate the format of their values"
