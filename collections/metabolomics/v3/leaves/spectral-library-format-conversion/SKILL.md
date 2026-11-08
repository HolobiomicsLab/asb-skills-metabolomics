---
name: spectral-library-format-conversion
description: Use when you hold spectral libraries in mixed .MSP/.MGF/.JSON/.CSV/.XML
  formats from OMSLs (GNPS, MoNA, MassBank, MS-Dial) and need one unified, metadata-harmonized
  library for dereplication or downstream annotation tools.
when_to_use_negative:
- Your data are already a unified, FragHub-standardized library rather than heterogeneous
  OMSL source files
- The input is raw LC-MS instrument data (e.g. mzML profiles from experiments) rather
  than a compiled reference spectral library
- You need harmonized collision energies across libraries — the paper explicitly did
  not address these due to around 70 different nonstandardized formats
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: FragHub
  role: Performs the parsing, field-name/value standardization, chemical-identifier
    normalization, SPLASH-key deduplication, and unified-format output of OMSL libraries
  repo: github.com/eMetaboHUB/FragHub
- name: Python
  role: Implementation language of the FragHub conversion code (MIT-licensed in the
    original release)
  repo: github.com/eMetaboHUB/FragHub
- name: MatchMS
  role: Python package in the MatchMS satellite ecosystem used for mass spectral processing
    alongside the conversion workflow
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/acs.analchem.4c02219
    title: 'FragHub: A Mass Spectral Library Data Integration Workflow'
  source_content_hash: sha256:ed71b54446d4ebffdcb456132353174d5091571a8bd5fdc15f41ce16c8694034
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/eMetaboHUB/FragHub
  tools:
  - FragHub
  - MatchMS
  - Python
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# spectral-library-format-conversion

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Converts and harmonizes mass spectral libraries from heterogeneous Open Mass Spectra Library (OMSL) formats (.MSP, .MGF, .JSON, .CSV, .XML) into a single unified, standardized FragHub output, so spectra from GNPS, MoNA, MassBank, and MS-Dial can be searched with tools like MS-Dial, MZmine, and Flash Entropy Search.

## When to use

Use when you hold spectral libraries in mixed .MSP/.MGF/.JSON/.CSV/.XML formats from OMSLs (GNPS, MoNA, MassBank, MS-Dial) and need one unified, metadata-harmonized library for dereplication or downstream annotation tools.

## When NOT to use

- Your data are already a unified, FragHub-standardized library rather than heterogeneous OMSL source files
- The input is raw LC-MS instrument data (e.g. mzML profiles from experiments) rather than a compiled reference spectral library
- You need harmonized collision energies across libraries — the paper explicitly did not address these due to around 70 different nonstandardized formats

## Inputs

- OMSL spectral library files in .MSP format (e.g. MassBank, MoNA exports)
- OMSL spectral library files in .MGF format (e.g. GNPS-tagged databases)
- OMSL spectral library files in .JSON format (e.g. MZmine exports)
- .CSV spectral tables separated by ';' or '\t' with quotechar '"' and a 'peaks' column
- OMSL spectral library files in .XML format

## Outputs

- Unified, standardized FragHub spectral library in user-selected output format(s), compatible with MS-Dial, MZmine, and Flash Entropy Search
- Spectra segregated by polarity (positive/negative), chromatographic mode (LC/GC), and acquisition type (experimental/in silico)
- 'update.json' file holding SPLASH keys of previously processed spectra for incremental updates
- Cleaned versions of the library files (do not delete files in the output directory after copying)

## How to apply

Parse each input library (.MSP, .MGF, .JSON, .CSV, .XML) as the initial FragHub workflow step; for CSV inputs, use ';' or tab separators with quotechar '"' and name the peak column 'peaks'. Standardize field names and values, then recalculate and normalize chemical identifiers (SMILES, InChI, InChIKey) and compute exact and average molecular masses with RDKit and internal dictionaries. Generate a SPLASH key per spectrum and record all SPLASH keys in the 'update.json' file so that on subsequent runs only new spectra are processed. Drop inconsistent spectra that lack SMILES and InChI, precursor m/z, or adduct information, then classify the remaining spectra by polarity (positive/negative), chromatographic mode (GC/LC-MS), and acquisition type (experimental/in silico; suffix '_insilico' or '_GC' filenames when not annotated). Finally select the desired output format(s) in the GUI and write the unified library; in the paper this refined 794,985 spectra / 35,673 unique identifiers into 602,744 spectra / 32,193 unique chemicals.

## Related tools

- **FragHub** (Performs the parsing, field-name/value standardization, chemical-identifier normalization, SPLASH-key deduplication, and unified-format output of OMSL libraries) — github.com/eMetaboHUB/FragHub
- **Python** (Implementation language of the FragHub conversion code (MIT-licensed in the original release)) — github.com/eMetaboHUB/FragHub
- **MatchMS** (Python package in the MatchMS satellite ecosystem used for mass spectral processing alongside the conversion workflow)

## Evaluation signals

- Each output spectrum carries recalculated, normalized SMILES, InChI, and InChIKey identifiers plus exact and average molecular masses
- Every processed spectrum has a SPLASH key recorded in 'update.json'; rerunning the workflow processes only new spectra
- Output spectra are partitioned by polarity (positive/negative), LC/GC mode, and experimental vs in silico acquisition, with no unclassified spectra
- On the paper's four OMSLs (GNPS, MoNA, MassBank, MS-Dial-VS17), conversion of 794,985 spectra / 35,673 unique chemical identifiers yields 602,744 spectra / 32,193 unique chemicals after cleaning and deduplication
- Output files load in downstream tools (MS-Dial, MZmine, Flash Entropy Search) without format errors

## Limitations

- All spectra lacking SMILES and InChI, precursor m/z, and adduct information are removed during FragHub processing
- Collision energy harmonization was not addressed because of around 70 different nonstandardized formats
- CSV inputs must use ';' or '\t' separators, quotechar '"', and a peak column named exactly 'peaks'; output-directory files must not be deleted after copying cleaned versions
- License changed from MIT to CC-BY-NC 4.0 (non-commercial restrictions), and MacOS/Linux support was removed in the 20_12_2024 release

## Evidence
