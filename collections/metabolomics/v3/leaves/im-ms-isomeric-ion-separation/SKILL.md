---
name: im-ms-isomeric-ion-separation
description: Use when processing LC-DTIM-MS mzML files of small molecules where isomeric
  ion species (e.g., protomers or conformers) must be separated in the IM dimension
  and DTCCS(N2) values assigned after RT confirmation of a primary ion.
when_to_use_negative:
- Input is not drift-tube IM-MS data (the tool is built for drift tube IM-MS measurements;
  e.g., TWIMS data would need a different calibration approach)
- No single-field CCS calibration coefficients (Tfix, Beta) are available for the
  data file, so arrival times cannot be converted to DTCCS(N2)
- The goal is untargeted feature finding without prior molecular formulae or confirmed
  RT windows — CCSfind is chemically informed and requires targeted formulae
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0153
- http://edamontology.org/topic_3361
tools:
- name: CCSfind
  role: Runs the RT-window adduct search and second peak detection in the arrival-time
    dimension, then converts arrival times to DTCCS(N2) via single-field calibration
  repo: https://github.com/sangeeta97/ccs_find.git
- name: Python 3.9
  role: Required runtime for processor.py and gui.py of CCSfind
  repo: https://github.com/sangeeta97/ccs_find.git
- name: conda
  role: Creates the Python 3.9 environment in which the CCSfind processor is launched
- name: numpy
  role: Dependency installed via requirements.txt supporting the CCSfind processing
- name: pandas
  role: Dependency installed via requirements.txt supporting the CCSfind result assembly
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1002/jms.5040
    title: 'CCSfind: A tool for chemically informed LC‐IM‐MS database building'
  source_content_hash: sha256:da6722101497e2d77abee6dcfc407af6e8f036b803f0ddfecde85b961f696e45
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
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0153
  - http://edamontology.org/topic_3361
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# im-ms-isomeric-ion-separation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

This skill covers the CCSfind workflow for resolving isomeric ion species (protomers, conformers) of small molecules in drift-tube LC-IM-MS data: after retention-time confirmation, targeted adduct ions are extracted over the confirmed RT window, a second peak-detection pass is run in the arrival-time (IM) dimension, and arrival times are converted to DTCCS(N2) using single-field calibration coefficients (Tfix, beta) with nitrogen as drift gas.

## When to use

Use when processing LC-DTIM-MS mzML files of small molecules where isomeric ion species (e.g., protomers or conformers) must be separated in the IM dimension and DTCCS(N2) values assigned after RT confirmation of a primary ion.

## When NOT to use

- Input is not drift-tube IM-MS data (the tool is built for drift tube IM-MS measurements; e.g., TWIMS data would need a different calibration approach)
- No single-field CCS calibration coefficients (Tfix, Beta) are available for the data file, so arrival times cannot be converted to DTCCS(N2)
- The goal is untargeted feature finding without prior molecular formulae or confirmed RT windows — CCSfind is chemically informed and requires targeted formulae

## Inputs

- Profile-data mzML file from LC-DTIM-MS (up to ~1 GB)
- List of molecular formulae to search
- Predicted/expected adduct ion species (positive and negative mode)
- Confirmed retention-time windows for primary ions
- Single-field CCS calibration coefficients (Tfix and Beta)
- Drift gas identity (e.g., nitrogen)

## Outputs

- Per-compound DTCCS(N2) values for primary ions and adduct ions
- Full mobility spectra / CCS distributions per compound entry
- SQL-format database entries (user-controlled output) and output workbook

## How to apply

Configure the CCSfind run via a config.ini file (launched with 'python processor.py -g config.ini' under Python 3.9, environment built with conda) listing the molecular formulae, expected adduct ions for positive and negative modes, confirmed RT windows, and the single-field CCS calibration coefficients Tfix and Beta with the drift gas identity (e.g., nitrogen). Parse the profile-mode mzML input and, once a targeted molecular formula's primary ion RT is confirmed (peak picking with isotopologue matching confirmation for A, A+1, A+2), search the expected adduct ions across the confirmed peak's RT width. Then perform a second peak-detection pass in the arrival-time (ion mobility) dimension for each confirmed primary ion and its adduct ions, which separates isomeric species such as protomers or conformers that coelute chromatographically. Convert each detected arrival time to DTCCS(N2) with the single-field external calibration (slope beta, intercept Tfix) and assemble per-compound CCS values plus full mobility spectra into a unified result set written as an Excel/SQL-format output. The rationale is that current analytical databases do not retain full IM-MS information, so the tool preserves mobility spectra and multiple ion species per compound entry, with results manually reviewed and annotated with experimental settings before committing to the database.

## Related tools

- **CCSfind** (Runs the RT-window adduct search and second peak detection in the arrival-time dimension, then converts arrival times to DTCCS(N2) via single-field calibration) — https://github.com/sangeeta97/ccs_find.git
- **Python 3.9** (Required runtime for processor.py and gui.py of CCSfind) — https://github.com/sangeeta97/ccs_find.git
- **conda** (Creates the Python 3.9 environment in which the CCSfind processor is launched)
- **numpy** (Dependency installed via requirements.txt supporting the CCSfind processing)
- **pandas** (Dependency installed via requirements.txt supporting the CCSfind result assembly)

## Examples

```
python processor.py -g config.ini
```

## Evaluation signals

- Output workbook is produced and DTCCS(N2) values are returned for all QC compounds whose adduct ions were found within the confirmed RT windows
- Each confirmed primary ion has an adduct-ion result set covering the expected adduct species searched over the confirmed RT width
- Coeluting protomer/conformer pairs appear as distinct mobility peaks after the IM-dimension second peak-detection pass
- Conversion check: DTCCS(N2) values reproduce from arrival times via the supplied single-field calibration (slope beta, intercept Tfix) with nitrogen drift gas
- Isotopologue matching confirmation (A, A+1, A+2) passed for peaks before RT/IM assignment

## Limitations

- Requires single-field CCS calibration coefficients (Tfix, Beta) specific to the data file; without them no DTCCS values can be computed
- Handles isomer separation only where chromatographically separated or resolved in the IM (arrival-time) dimension; unresolved isomeric ions may not separate
- Profile-mode mzML input from LC-DTIM-MS is required; the compulsory inputs (formulae list, adducts, calibration coefficients, drift gas, primary ion species) must all be supplied
- Processing time scales linearly with mzML file size and number of input molecular formulae

## Evidence
