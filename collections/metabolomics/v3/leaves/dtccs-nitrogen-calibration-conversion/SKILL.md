---
name: dtccs-nitrogen-calibration-conversion
description: Use when processing LC-DTIM-MS mzML data with a confirmed primary-ion
  RT for a targeted molecular formula and single-field CCS calibration coefficients
  (Tfix, Beta) for nitrogen drift gas are available, and DTCCS(N2) values plus adduct-ion
  mobility spectra are needed.
when_to_use_negative:
- Input is not drift tube IM-MS data (e.g., TIMS or cyclic-IMS), since single-field
  Tfix/Beta calibration with a drift gas does not apply.
- No single-field CCS calibration coefficients (Tfix, Beta) are available for the
  data file, so arrival times cannot be converted to DTCCS(N2).
- The input is already a processed feature table or existing CCS database rather than
  raw profile-mode mzML from LC-DTIM-MS.
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: CCSfind
  role: Tool performing adduct-ion RT-window search, IM-dimension peak detection,
    and DTCCS(N2) conversion from parsed mzML files
  repo: https://github.com/sangeeta97/ccs_find.git
- name: Python 3.9
  role: Runtime environment (conda/pip) required to run processor.py with config.ini
  repo: https://github.com/sangeeta97/ccs_find.git
- name: numpy
  role: Dependency used in the processing pipeline installed via requirements.txt
  repo: https://github.com/sangeeta97/ccs_find.git
- name: pandas
  role: Dependency used for assembling the result table installed via requirements.txt
  repo: https://github.com/sangeeta97/ccs_find.git
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1002/jms.5040
    title: 'CCSfind: A tool for chemically informed LC‐IM‐MS database building'
  source_content_hash: sha256:9c3aa7fc77159004bd2611f2c828bc55776c0f93ccbe30bcea97cc8722437aa3
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
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# dtccs-nitrogen-calibration-conversion

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Second-stage extraction in CCSfind: after the primary ion's retention time is confirmed, search expected adduct ions within the confirmed RT window, run peak detection in the ion-mobility (arrival time) dimension, and convert arrival times to DTCCS(N2) using single-field calibration coefficients (Tfix and Beta) with nitrogen as drift gas.

## When to use

Use when processing LC-DTIM-MS mzML data with a confirmed primary-ion RT for a targeted molecular formula and single-field CCS calibration coefficients (Tfix, Beta) for nitrogen drift gas are available, and DTCCS(N2) values plus adduct-ion mobility spectra are needed.

## When NOT to use

- Input is not drift tube IM-MS data (e.g., TIMS or cyclic-IMS), since single-field Tfix/Beta calibration with a drift gas does not apply.
- No single-field CCS calibration coefficients (Tfix, Beta) are available for the data file, so arrival times cannot be converted to DTCCS(N2).
- The input is already a processed feature table or existing CCS database rather than raw profile-mode mzML from LC-DTIM-MS.

## Inputs

- Profile-data mzML file from LC-DTIM-MS (up to ~1 GB)
- List of molecular formulae to search for
- Single-field CCS calibration coefficients (Tfix and Beta) for the data file
- Drift gas identity (e.g., nitrogen)
- Primary ion species and expected adduct ions (positive and negative modes)
- config.ini with confirmed RT windows and extraction parameters

## Outputs

- DTCCS(N2) values per compound, primary ion, and adduct ion
- Full mobility spectra (CCS distributions) per ion species, including resolved protomers/conformers
- Unified result set / SQL-format database entries (workbook output via XlsxWriter)

## How to apply

Configure the CCSfind run via config.ini with the molecular formulae, expected adduct ions (positive and negative mode), confirmed RT windows, single-field calibration coefficients (Tfix and Beta), and drift gas identity (e.g., nitrogen), then launch 'python processor.py -g config.ini' under Python 3.9. After parsing the profile-mode mzML files and confirming the primary ion via isotopologue matching (A, A+1, A+2), search the expected adduct ions across the confirmed peak's retention-time width, with an adjustable mass extraction window and adjustable expected LC peak width/abundance. For each confirmed primary ion and its adduct ions, perform a second peak-detection pass in the arrival-time (IM) dimension, which also resolves IM-separated isomeric ions such as protomers or conformers. Convert each detected arrival time to DTCCS(N2) using the single-field external calibration (Tfix, beta) and assemble per-compound CCS values and full mobility spectra into the unified result set. Results are then manually reviewed and annotated with experimental settings before committing to the SQL-format database.

## Related tools

- **CCSfind** (Tool performing adduct-ion RT-window search, IM-dimension peak detection, and DTCCS(N2) conversion from parsed mzML files) — https://github.com/sangeeta97/ccs_find.git
- **Python 3.9** (Runtime environment (conda/pip) required to run processor.py with config.ini) — https://github.com/sangeeta97/ccs_find.git
- **numpy** (Dependency used in the processing pipeline installed via requirements.txt) — https://github.com/sangeeta97/ccs_find.git
- **pandas** (Dependency used for assembling the result table installed via requirements.txt) — https://github.com/sangeeta97/ccs_find.git

## Examples

```
conda create --name ccs_find python=3.9; conda activate ccs_find; pip install -r requirements.txt; python processor.py -g config.ini
```

## Evaluation signals

- Output workbook is produced and DTCCS(N2) values are returned for all targeted compounds whose adduct ions were found within the confirmed RT windows.
- Every reported DTCCS(N2) traces back to a detected arrival time converted with the supplied Tfix and Beta coefficients for nitrogen drift gas.
- Isomeric ions (protomers/conformers) separated in the IM dimension appear as distinct entries with their own arrival times and CCS values.
- Isotopologue matching confirmation (A, A+1, A+2) passed for each primary ion before adduct search and IM peak detection.
- Computational time scales linearly with mzML file size and number of molecular formulae (files up to 1 GB processed in under 2 minutes on a 32 GB RAM desktop).

## Limitations

- Requires profile-data mzML from LC-DTIM-MS and Python 3.9 specifically; other runtimes are not supported.
- Single-field calibration is drift-gas-specific — coefficients (Tfix, Beta) obtained with nitrogen cannot be reused for other drift gases.
- Extraction quality depends on user-set parameters: mass extraction window for isotopologue confirmation, expected LC peak width, and expected peak abundance (noisy vs well-defined).
- IM-dimension peak detection must disentangle isomeric species (protomers, conformers); unresolved mobility peaks may merge contributions.

## Evidence
