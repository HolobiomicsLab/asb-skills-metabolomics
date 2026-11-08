---
name: ion-mobility-arrival-time-peak-detection
description: Use when you have drift tube IM-MS mzML data, a confirmed retention time
  for a molecular formula's primary ion, and single-field CCS calibration coefficients
  (Tfix, Beta) with a known drift gas, and you need IM peaks and DTCCS(N2) values
  for adduct ion species.
when_to_use_negative:
- Data are not drift tube IM-MS profile measurements in mzML format (e.g., TIMS or
  non-IM LC-MS data), since single-field Tfix/Beta calibration is drift-tube specific
- No single-field CCS calibration coefficients (Tfix and Beta) or drift gas identity
  are available for the data file
- The retention time of the target molecular formula's primary ion has not yet been
  confirmed, so there is no RT window in which to search for adduct ions
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: CCSfind
  role: Performs the RT-window adduct search, the second peak detection in the arrival-time
    dimension, and the arrival-time to DTCCS(N2) conversion
  repo: https://github.com/sangeeta97/ccs_find.git
- name: Python 3.9
  role: Required runtime to run processor.py/config.ini-driven analysis
- name: numpy
  role: Dependency installed via requirements.txt used in the CCSfind processing pipeline
- name: pandas
  role: Dependency installed via requirements.txt used in the CCSfind processing pipeline
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1002/jms.5040
    title: 'CCSfind: A tool for chemically informed LC‐IM‐MS database building'
  source_content_hash: sha256:7d44da2d2fb3e720e3233270b7688efe2c6491121504e9c487ca7a6f79799213
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
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# ion-mobility-arrival-time-peak-detection

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

CCSfind's second peak-detection pass in the arrival-time (ion mobility) dimension, performed for each confirmed primary ion and its expected adduct ions, followed by conversion of arrival times to DTCCS(N2) via single-field calibration coefficients (Tfix, beta) with nitrogen drift gas.

## When to use

Use when you have drift tube IM-MS mzML data, a confirmed retention time for a molecular formula's primary ion, and single-field CCS calibration coefficients (Tfix, Beta) with a known drift gas, and you need IM peaks and DTCCS(N2) values for adduct ion species.

## When NOT to use

- Data are not drift tube IM-MS profile measurements in mzML format (e.g., TIMS or non-IM LC-MS data), since single-field Tfix/Beta calibration is drift-tube specific
- No single-field CCS calibration coefficients (Tfix and Beta) or drift gas identity are available for the data file
- The retention time of the target molecular formula's primary ion has not yet been confirmed, so there is no RT window in which to search for adduct ions

## Inputs

- Profile-mode mzML file from LC-DTIM-MS (up to 1 GB)
- List of molecular formulae to be searched in the mzML file
- Confirmed retention-time windows for the primary ions
- Expected adduct ion species (positive and/or negative mode; multiply charged ions optional)
- Single-field CCS calibration coefficients (Tfix and Beta)
- Identity of the drift gas (e.g., nitrogen)

## Outputs

- Per-ion arrival-time (IM dimension) peak detections, including resolved protomer/conformer mobility peaks
- DTCCS(N2) values computed from the single-field calibration (Tfix, Beta)
- Unified per-compound result set (CCS values and mobility spectra) written as an output table/workbook for database entries

## How to apply

First confirm the retention time of the targeted molecular formula's primary ion, then configure the CCSfind run by editing config.ini with the QC-compound molecular formulae, expected adduct ions, and confirmed RT windows, and launch it via 'python processor.py -g config.ini' under Python 3.9. CCSfind parses the open-source mzML files and targets the predicted adduct ion species within each compound's confirmed retention-time window, applying isotopologue matching confirmation (A, A+1, A+2) after peak picking. For each confirmed primary ion and its corresponding adduct ions, perform a second peak-detection pass in the arrival-time (IM) dimension, which also resolves IM-separated isomeric ions such as protomers or conformers. Convert each detected arrival time to DTCCS(N2) using the single-field external calibration coefficients (Tfix, Beta) supplied for the drift gas, e.g. nitrogen. Assemble the per-compound CCS values and mobility spectra into the unified result set written as the output table/workbook.

## Related tools

- **CCSfind** (Performs the RT-window adduct search, the second peak detection in the arrival-time dimension, and the arrival-time to DTCCS(N2) conversion) — https://github.com/sangeeta97/ccs_find.git
- **Python 3.9** (Required runtime to run processor.py/config.ini-driven analysis)
- **numpy** (Dependency installed via requirements.txt used in the CCSfind processing pipeline)
- **pandas** (Dependency installed via requirements.txt used in the CCSfind processing pipeline)

## Examples

```
conda create --name ccs_find python=3.9; pip install -r requirements.txt; python processor.py -g config.ini
```

## Evaluation signals

- Output workbook/table is produced and DTCCS values are returned for all eight QC compounds whose adduct ions were found within the confirmed RT windows
- Detected arrival times convert to DTCCS(N2) using the supplied Tfix and Beta calibration coefficients with the declared drift gas
- Isomeric IM species (protomers/conformers) of the same ion species are separated as distinct mobility peaks
- Isotopologue matching confirmation (A, A+1, A+2) passed after peak picking for the targeted ions
- mzML files of up to 1 GB processed in less than 2 minutes on a desktop with 32 GB RAM, with linear scaling in file size and number of formulae

## Limitations

- Requires profile-data mzML input from LC-DTIM-MS; not applicable to other IM-MS platforms
- Single-field calibration (Tfix, Beta) is specific to the drift gas (e.g., nitrogen) and the data file considered
- Separate parameters must be tuned for noisy vs well-defined peaks and expected LC peak width
- Computational time scales linearly with input mzML file size and number of input molecular formulae

## Evidence
