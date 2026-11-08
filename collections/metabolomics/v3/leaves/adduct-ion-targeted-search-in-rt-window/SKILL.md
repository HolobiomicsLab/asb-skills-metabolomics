---
name: adduct-ion-targeted-search-in-rt-window
description: Use when the primary ion of a molecular formula has a confirmed RT window
  in a profile LC-DTIM-MS mzML file and you need to screen additional adduct ions
  (positive/negative mode), extract IM peaks, and derive DTCCS(N2) values.
when_to_use_negative:
- The primary ion's retention time has not yet been confirmed, so no RT window exists
  to constrain the adduct search
- Input data are not profile LC-DTIM-MS mzML files (e.g., centroided or non-IM-MS
  data)
- Single-field CCS calibration coefficients (Tfix, beta) or drift gas identity are
  unavailable, so arrival times cannot be converted to DTCCS(N2)
edam_operation: http://edamontology.org/operation_3221
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: CCSfind
  role: Performs the adduct-ion targeted search in the confirmed RT window, IM-dimension
    peak detection, and DTCCS(N2) calibration
  repo: https://github.com/sangeeta97/ccs_find.git
- name: Python 3.9
  role: Required runtime environment (conda or pip) to execute processor.py
- name: pandas
  role: Dependency for assembling and writing the result table
- name: numpy
  role: Dependency for numerical processing of peak/arrival-time data
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1002/jms.5040
    title: 'CCSfind: A tool for chemically informed LC‐IM‐MS database building'
  source_content_hash: sha256:9013200b3d3f1db4515477f281f0cd06be19562d915b6b72db95e5edf081894b
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/sangeeta97/ccs_find
  repo_url: https://github.com/sangeeta97/ccs_find.git
  tools:
  - CCSfind
  - Python
  - Python 3.9
  - conda
  - numpy
  - pandas
  edam_operation: http://edamontology.org/operation_3221
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# adduct-ion-targeted-search-in-rt-window

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

After a targeted molecular formula's primary ion retention time is confirmed, CCSfind searches expected adduct ion species within the confirmed RT window and then performs a second peak-detection pass in the ion-mobility (arrival-time) dimension to build the full per-compound result set, converting arrival times to DTCCS(N2) with single-field calibration coefficients (Tfix, beta).

## When to use

Use when the primary ion of a molecular formula has a confirmed RT window in a profile LC-DTIM-MS mzML file and you need to screen additional adduct ions (positive/negative mode), extract IM peaks, and derive DTCCS(N2) values.

## When NOT to use

- The primary ion's retention time has not yet been confirmed, so no RT window exists to constrain the adduct search
- Input data are not profile LC-DTIM-MS mzML files (e.g., centroided or non-IM-MS data)
- Single-field CCS calibration coefficients (Tfix, beta) or drift gas identity are unavailable, so arrival times cannot be converted to DTCCS(N2)

## Inputs

- Profile data mzML file from LC-DTIM-MS (up to 1 GB)
- List of molecular formulae to search
- Primary ion species to search for
- Expected adduct ions (positive and negative mode)
- Confirmed retention-time windows per compound
- Single-field CCS calibration coefficients (Tfix and beta)
- Drift gas identity (e.g., nitrogen)

## Outputs

- Unified result set of per-compound CCS values (DTCCS N2) for primary and adduct ions
- Full mobility spectra (CCS distributions) per targeted ion
- SQL-format database entries / output workbook (xlsx via XlsxWriter)

## How to apply

Configure config.ini with the target molecular formulae, expected adduct ions, and confirmed RT windows, then run CCSfind via processor.py under Python 3.9. Parse the profile mzML files and target the predicted adduct ion species strictly within each compound's confirmed retention-time window. For each confirmed primary ion and its corresponding adduct ions, run a second peak-detection pass in the arrival-time (ion mobility) dimension, which also separates isomeric species such as protomers or conformers. Convert each detected arrival time to DTCCS(N2) using the single-field external calibration coefficients (Tfix and beta) for the drift gas (e.g., nitrogen). Apply isotopologue matching confirmation (A, A+1, A+2) as the filter step after peak picking, then assemble the per-compound CCS values and full mobility spectra into a unified result set with user-controlled SQL-format database entries.

## Related tools

- **CCSfind** (Performs the adduct-ion targeted search in the confirmed RT window, IM-dimension peak detection, and DTCCS(N2) calibration) — https://github.com/sangeeta97/ccs_find.git
- **Python 3.9** (Required runtime environment (conda or pip) to execute processor.py)
- **pandas** (Dependency for assembling and writing the result table)
- **numpy** (Dependency for numerical processing of peak/arrival-time data)

## Examples

```
python processor.py -g config.ini
```

## Evaluation signals

- The output workbook is produced (xlsx) with per-compound entries
- DTCCS(N2) values are returned for all targeted compounds (e.g., all eight QC compounds) whose adduct ions were found within the confirmed RT windows
- Each primary ion and adduct ion has an extracted arrival-time (mobility) peak from the second peak-detection pass
- Isomeric ions (protomers/conformers) are resolved as separate IM peaks where chromatographically coeluting
- Isotopologue matching confirmation (A, A+1, A+2) passed for accepted peaks

## Limitations

- Requires prior confirmation of the primary ion RT window; the adduct search is constrained to that window
- Single-field DTCCS(N2) conversion depends on correct Tfix, beta coefficients and drift gas identity for the data file
- Performance scales linearly with mzML file size and number of input formulae; up to 1 GB files process in <2 min only on a 32 GB RAM desktop
- Isomeric species must be separable in IM (protomers/conformers); CCSfind cannot resolve IM-inseparable isobars

## Evidence

- [abstract] CCSfind allows predicted ion species to be chosen for input chemical formulae, which are then targeted by CCSfind after parsing open source mzML input files: "CCSfind allows predicted ion species to be chosen for input chemical formulae, which are then targeted by CCSfind after parsing open source mzML input files"
- [methods] python processor.py config invocation: "python processor.py [-help] 
    [-g config.ini, this file has all parameter to run the analysis, can be changed as per analysis criteria]"
- [readme] Supports screening of additional adducts ions for both positive and negative modes.: "Supports screening of additional adducts ions for both positive and negative modes."
- [readme] Compulsory inputs include calibration coefficients and drift gas: "The single-field CCS calibration coefficients (Tfix and Beta) for the data file considered."
- [intro] CCSfind handles isomeric ion separation: "CCSfind can handle both chromatographically separated isomers and IM separation of isomeric ions (e.g., "protomers" or conformers of the same ion species)"
