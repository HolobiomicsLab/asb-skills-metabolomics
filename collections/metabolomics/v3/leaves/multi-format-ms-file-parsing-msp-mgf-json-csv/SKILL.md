---
name: multi-format-ms-file-parsing-msp-mgf-json-csv
description: Use when ingesting mass spectral library records from multiple OMSL sources
  in differing file formats (.MSP, .MGF, .JSON, .CSV, .XML) whose field names and
  values must be harmonized before integration or annotation workflows.
when_to_use_negative:
- The input is already a unified, FragHub-processed library whose fields are already
  in the harmonized schema — re-parsing would risk breaking the naming.
- The goal is processing raw instrument acquisition files (e.g. mzML/Thermo raw) rather
  than spectral library records — FragHub handles OMSL library formats, not raw data.
- 'The source file encodes only what FragHub cannot use: spectra lacking both SMILES
  and InChI, precursor m/z and adduct information are removed during processing anyway.'
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3360
tools:
- name: FragHub
  role: Workflow that performs the multi-format parsing and field-name mapping into
    a unified format
  repo: github.com/eMetaboHUB/FragHub
- name: Python
  role: Implementation language of the FragHub parser (MIT-licensed code in the article;
    later releases CeCILL)
  repo: github.com/eMetaboHUB/FragHub
- name: MZmine
  role: Source of spectral library generation module outputs (.JSON, .MGF, .MSP) that
    the parser must accept
- name: MS-Dial
  role: Source library format (.MSP/MassBank records) consumed and target software
    for compatibility
- name: MatchMS
  role: Python package for mass spectra handling referenced in the intro as related
    infrastructure
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/acs.analchem.4c02219
    title: 'FragHub: A Mass Spectral Library Data Integration Workflow'
  source_content_hash: sha256:9303630588ba4f55478790326769b17653d3e64b15ece76afcc8b87d0cebf528
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/eMetaboHUB/FragHub
  tools:
  - Flash Entropy Search
  - FragHub
  - MS-Dial
  - MZmine
  - MatchMS
  - Python
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3360
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# multi-format-ms-file-parsing-msp-mgf-json-csv

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Parse heterogeneous open mass spectral library (OMSL) files in .MSP, .MGF, .JSON, .CSV and .XML formats (e.g. GNPS, MoNA, MassBank, MS-Dial exports) and map their field names/values to a unified, GNPS-compatible naming scheme so downstream tools like MS-Dial, MZmine and Flash Entropy Search can consume the result.

## When to use

Use when ingesting mass spectral library records from multiple OMSL sources in differing file formats (.MSP, .MGF, .JSON, .CSV, .XML) whose field names and values must be harmonized before integration or annotation workflows.

## When NOT to use

- The input is already a unified, FragHub-processed library whose fields are already in the harmonized schema — re-parsing would risk breaking the naming.
- The goal is processing raw instrument acquisition files (e.g. mzML/Thermo raw) rather than spectral library records — FragHub handles OMSL library formats, not raw data.
- The source file encodes only what FragHub cannot use: spectra lacking both SMILES and InChI, precursor m/z and adduct information are removed during processing anyway.

## Inputs

- GNPS .MGF library files
- MoNA .MSP, .SDF and .JSON records
- MassBank records
- MS-Dial-VS17 library exports (.MSP)
- MZmine3 spectral library generation module output (.JSON/.CSV)
- In-house CSV files with ';' or tab separators, quotechar '"', and a 'peaks' column

## Outputs

- Parsed spectra with field names mapped to the GNPS-compatible unified schema
- Per-format field name/value mapping records following Table 1 conventions
- Parsed spectra objects ready for metadata cleaning, SPLASH key generation and filtering

## How to apply

Parse the source files field-by-field according to the per-format field-name/value table (Table 1), then apply a mapping dictionary that translates source-specific keys to GNPS-compatible naming so outputs remain usable with MS-Dial, MZmine and Flash Entropy Search. For CSV inputs, require semicolon or tab separators with quotechar '"' and a 'peaks' column formatted per the accepted conventions. Special-case files that contain only in-silico or GC spectra not annotated per-record: suffix the filename with '_insilico' or '_GC' (e.g. 'UNPD_ISDB_R_p01_insilico.mgf', 'GMD_20111121_MDN35_ALK_MSP_GC.msp') so classification is preserved. The rationale is that OMSLs 'encompass a variety of file structures', so a single parser per format plus a shared dictionary is what enables concatenation into one unified library.

## Related tools

- **FragHub** (Workflow that performs the multi-format parsing and field-name mapping into a unified format) — github.com/eMetaboHUB/FragHub
- **Python** (Implementation language of the FragHub parser (MIT-licensed code in the article; later releases CeCILL)) — github.com/eMetaboHUB/FragHub
- **MZmine** (Source of spectral library generation module outputs (.JSON, .MGF, .MSP) that the parser must accept)
- **MS-Dial** (Source library format (.MSP/MassBank records) consumed and target software for compatibility)
- **MatchMS** (Python package for mass spectra handling referenced in the intro as related infrastructure)

## Examples

```
FragHub GUI → First tab: 'Select single or multiple .json, .csv, .msp, or .mgf files' (or run the FragHub executable from the install directory; Python 3.12 code at github.com/eMetaboHUB/FragHub)
```

## Evaluation signals

- All source field keys from each input format (.MSP, .MGF, .JSON, .CSV, .XML) resolve to the mapping dictionary — no unmapped keys remain in the parsed output.
- Spectra counts per source match published baselines: GNPS 63,935 spectra / 13,507 compounds; MoNA 190,359 spectra / 21,839 compounds; MassBank 164,261 records / 8,358 compounds; MS-Dial-VS17 376,430 spectra / 22,282 compounds.
- CSV inputs parsed only when separators are ';' or tab with quotechar '"' and a 'peaks' column exists.
- Filename-suffixed '_insilico' or '_GC' files are correctly classified as predicted-origin or GC-MS spectra respectively.
- Parsed records retain peaks, precursor m/z, ion mode and identifier fields well enough that downstream cleaning can compute SPLASH keys and InChI/SMILES-derived masses.

## Limitations

- Harmonization of collision energies was not addressed due to around 70 different nonstandardized measures across sources.
- Placeholder values such as 'RT: 0.0' or 'adduct: unknown' must be mapped to 'UNKNOWN' during downstream cleaning — raw parsing alone leaves them inconsistent.
- Version/length caveats: source field tables must be updated when OMSL exporters change their schemas; no software version numbers are stated for RDKit, PubChem, PeakForest in the article.
- macOS and Linux support was removed in the 20_12_2024 release of the packaged executable (parse capability via Python code remains).

## Evidence
