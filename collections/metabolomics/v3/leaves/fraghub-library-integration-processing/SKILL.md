---
name: fraghub-library-integration-processing
description: Use when multiple .MSP, .MGF, .JSON, .CSV or .XML open mass spectral
  libraries (or an MZmine-generated in-house library) must be concatenated, metadata-harmonized,
  filtered, and exported for dereplication tools like MS-Dial, MZmine, or Flash Entropy
  Search.
when_to_use_negative:
- Input is already a FragHub-standardized output directory with update.json SPLASH
  keys — reprocessing it from scratch would require resetting the project first.
- Spectra lack both InChI and SMILES, or lack a valid precursor m/z — these inconsistent
  spectra are automatically removed, so the skill cannot recover unidentifiable entries.
- The goal is to harmonize collision energy annotations across libraries — FragHub
  explicitly does not harmonize collision energies because ~70 nonstandardized formats
  exist.
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: FragHub
  role: GUI tool that parses, filters, standardizes, deduplicates and exports the
    integrated mass spectral library; installed per-OS from Zenodo (Windows x64, Linux
    x64, macOS arm64 installers)
  repo: https://github.com/eMetaboHUB/FragHub
- name: MZmine
  role: Source of in-house spectral library generation module output that FragHub
    accepts as input for compatibility; standardized outputs are also compatible back
    into MZmine
- name: Python
  role: Implementation language of the open FragHub code (MIT-licensed in the paper;
    executable installs provided for non-programmers)
  repo: https://github.com/eMetaboHUB/FragHub
- name: RDKit
  role: Standardizes chemical identifiers (SMILES, InChI, InChIKey) and calculates
    exact and average molecular masses during metadata cleaning
- name: MS-Dial
  role: Downstream analysis software consuming FragHub-standardized output spectra;
    MS-Dial-VS17 was also one of the four integrated input libraries
- name: Flash Entropy Search
  role: Downstream search tool compatible with FragHub-standardized spectra; its entropy
    score is also used internally by the remove_spectrum_under_entropy_score filter
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1021/acs.analchem.4c02219
    title: 'FragHub: A Mass Spectral Library Data Integration Workflow'
  source_content_hash: sha256:374255814970a5d5b49673d995f2fd141e59eaf580e2941bf6fe5b07a98dbca5
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/eMetaboHUB/FragHub
  repo_url: https://github.com/eMetaboHUB/FragHub
  tools:
  - Flash Entropy Search
  - FragHub
  - MS-Dial
  - MZmine
  - Python
  - RDKit
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# fraghub-library-integration-processing

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Method for converting open or in-house mass spectral libraries (e.g. GNPS .MGF, MoNA .MSP/.SDF/.JSON, MassBank, MS-Dial exports, MZmine spectral-library-generation output) into a single FragHub-standardized, filtered, SPLASH-keyed library harmonized by RDKit and PSI-MS controlled vocabulary, segregated by ionization mode, LC/GC, and experimental vs in-silico origin.

## When to use

Use when multiple .MSP, .MGF, .JSON, .CSV or .XML open mass spectral libraries (or an MZmine-generated in-house library) must be concatenated, metadata-harmonized, filtered, and exported for dereplication tools like MS-Dial, MZmine, or Flash Entropy Search.

## When NOT to use

- Input is already a FragHub-standardized output directory with update.json SPLASH keys — reprocessing it from scratch would require resetting the project first.
- Spectra lack both InChI and SMILES, or lack a valid precursor m/z — these inconsistent spectra are automatically removed, so the skill cannot recover unidentifiable entries.
- The goal is to harmonize collision energy annotations across libraries — FragHub explicitly does not harmonize collision energies because ~70 nonstandardized formats exist.

## Inputs

- Open mass spectral library files in .MSP, .MGF, .JSON, .CSV (semicolon- or tab-separated, quoted, 'peaks' column), or .XML formats, e.g. GNPS .MGF, MoNA .MSP/.SDF/.JSON, MassBank records, MS-Dial-VS17 exports
- MZmine spectral library generation module output (following the MZmine tutorial in Supporting Information S6 for FragHub compatibility)
- FragHub output directory that already contains update.json, for incremental reprocessing of newly added spectra

## Outputs

- Unified-format standardized spectra (.MSP and .JSON among other selectable output formats) compatible with MS-Dial, MZmine, and Flash Entropy Search
- Libraries segregated by ionization mode (positive/negative), chromatography (LC/GC), and acquisition type (experimental/predicted in-silico)
- update.json SPLASH-key repository enabling incremental processing; de novo fragment formula annotations (optional)
- Refined, deduplicated library of unique chemicals (e.g. 794,985 input spectra reduced to 602,744 spectra for 32,193 unique chemicals in the paper's four-library integration)

## How to apply

Feed the GUI first tab with one or many .json, .csv, .msp, or .mgf files and point the second tab at a persistent output directory where FragHub records SPLASH keys in 'update.json' so later updates only process new spectra. On the filter tab enable user-defined spectrum filters and tune their parameters: check_minimum_peak_requiered (minimum peak count), remove_peak_above_precursormz (peaks > precursor m/z + 5 Da), reduce_peak_list (cap peak list by intensity), normalize_intensity (scale to maximum), keep_mz_in_range (mz_from to mz_to), check_minimum_of_high_peaks_requiered (peaks above an intensity_percent threshold), and remove_spectrum_under_entropy_score (minimum entropy score); optionally enable de novo fragment formula calculation. FragHub then standardizes identifiers with RDKit (SMILES/InChI/InChIKey recalculation, exact and average masses), harmonizes instrument and ionization terms against HUPO PSI-MS controlled vocabulary, generates SPLASH keys, deduplicates by InChIKey and fragment list, and segregates spectra by polarity (positive/negative), chromatographic mode (LC/GC), and acquisition type (experimental/in-silico) before exporting in the chosen output formats. CSV inputs must be ';'- or tab-separated with quotechar '"' and a 'peaks' column; files containing only in-silico or GC spectra must be suffixed '_insilico' or '_GC' if not otherwise specified. The rationale is that identical chemistry across GNPS, MoNA, MassBank and MS-Dial-VS17 differs in field names, values and formats, so integration and normalization are required before downstream annotation.

## Related tools

- **FragHub** (GUI tool that parses, filters, standardizes, deduplicates and exports the integrated mass spectral library; installed per-OS from Zenodo (Windows x64, Linux x64, macOS arm64 installers)) — https://github.com/eMetaboHUB/FragHub
- **MZmine** (Source of in-house spectral library generation module output that FragHub accepts as input for compatibility; standardized outputs are also compatible back into MZmine)
- **Python** (Implementation language of the open FragHub code (MIT-licensed in the paper; executable installs provided for non-programmers)) — https://github.com/eMetaboHUB/FragHub
- **RDKit** (Standardizes chemical identifiers (SMILES, InChI, InChIKey) and calculates exact and average molecular masses during metadata cleaning)
- **MS-Dial** (Downstream analysis software consuming FragHub-standardized output spectra; MS-Dial-VS17 was also one of the four integrated input libraries)
- **Flash Entropy Search** (Downstream search tool compatible with FragHub-standardized spectra; its entropy score is also used internally by the remove_spectrum_under_entropy_score filter)

## Examples

```
# Windows: run the installed executable, then via the GUI: load GNPS.mgf + MoNA.msp + MassBank.json, set filters (remove_peak_above_precursormz, normalize_intensity), enable '_insilico' suffix handling, export .MSP/.JSON
FragHub.exe
```

## Evaluation signals

- Output directory contains segregated subfolders for polarity (positive/negative), chromatographic mode (LC/GC), and acquisition type (experimental vs in-silico), with SPLASH keys recorded in update.json
- Every retained spectrum has SMILES and InChI (or InChIKey) and valid precursor m/z and adduct information; spectra lacking these were removed
- Spectrum count after processing is less than the summed input count (paper case: 794,985 input spectra → 602,744 output spectra across 32,193 unique chemicals from 35,673 input identifiers), demonstrating deduplication by InChIKey and fragment list
- Output spectra conform to GNPS naming conventions and load in MS-Dial, MZmine, and Flash Entropy Search without format errors
- On a second run with added spectra, only new SPLASH keys are processed — the previously processed entries in update.json are skipped, so runtime scales with novelty, not library size

## Limitations

- Collision energy harmonization is not addressed: around 70 different nonstandardized collision-energy formats exist across OMSLs, so CE values remain inconsistent across the integrated library.
- Redundancy across OMSLs can cause inconsistencies for dot product and reverse dot product scoring systems, which are highly sensitive to fragment number and intensity when the same compound appears multiple times.
- OMSL compound coverage (32,193 unique chemicals) is low relative to the estimated millions of natural products; chemical class disparity with natural product catalogues necessitates orthogonal strategies.
- Do not delete files in the selected output directory after copying cleaned versions — update.json and prior outputs must persist for incremental processing.
- All spectra lacking SMILES and InChI, precursor m/z, and adduct information are silently removed; in-silico-only or GC-only files must be filename-suffixed '_insilico' or '_GC' if not flagged in metadata.
- License changed from MIT to CC-BY-NC 4.0 (per repo discussion), and the 20_12_2024 release dropped macOS and Linux support — check the release notes for your OS.
- CSV inputs must use ';'- or tab-separation with quotechar '"' and a column named 'peaks'; other CSV dialects will fail parsing.

## Evidence
