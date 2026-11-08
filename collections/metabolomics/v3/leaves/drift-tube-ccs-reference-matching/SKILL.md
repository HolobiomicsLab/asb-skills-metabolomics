---
name: drift-tube-ccs-reference-matching
description: Use when processing traveling-wave IM-MS (e.g. Waters SYNAPT XS) calibration
  data (.mzML calibrant files) to derive class-specific power-law calibration parameters
  anchored to drift-tube CCS references, prior to calibrating unknown TWCCSN2 features.
when_to_use_negative:
- Calibration arrival times are already processed and tabulated — use MOCCal's UserDT
  executable path instead of the DEIMoS-based RawDT workflow
- Data were acquired in negative ionization mode — MOCCal is not yet extended to negative
  ionization mode data sets
- Features belong to chemical classes outside small molecules, lipids, and peptides,
  or the instrument is not a TWIM-MS platform
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: MOCCal
  role: Python application that loads calibrant data, optimizes class-specific power-law
    regression parameters, and writes the calibration-parameter files
  repo: https://github.com/HinesLab/MOCCal
- name: DEIMoS
  role: Used (version 1.3.2) to extract calibrant arrival-time data from .mzML files
    and fit each arrival-time distribution to a Gaussian whose apex becomes the arrival
    time
  repo: http://github.com/pnnl/deimos
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/acs.analchem.3c04290
    title: 'MOCCal: A Multiomic CCS Calibrator for Traveling Wave Ion Mobility Mass
      Spectrometry'
  source_content_hash: sha256:80b1dc6ecc6892f457ccfb65de984c7ab730c4bb8763a2ee05c9f3c64b506567
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/HinesLab/MOCCal
  tools:
  - DEIMoS
  - MOCCal
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7857142857142857
n_claims: 14
---

# drift-tube-ccs-reference-matching

## Summary

Fits class-specific power-law TWIM CCS calibration curves by regressing calibrant arrival times (Gaussian-fit apexes extracted with DEIMoS) against reference DTCCSN2 values from the Unified CCS Compendium, yielding one parameter set per biomolecular class (small molecules, lipids, peptides).

## When to use

Use when processing traveling-wave IM-MS (e.g. Waters SYNAPT XS) calibration data (.mzML calibrant files) to derive class-specific power-law calibration parameters anchored to drift-tube CCS references, prior to calibrating unknown TWCCSN2 features.

## When NOT to use

- Calibration arrival times are already processed and tabulated — use MOCCal's UserDT executable path instead of the DEIMoS-based RawDT workflow
- Data were acquired in negative ionization mode — MOCCal is not yet extended to negative ionization mode data sets
- Features belong to chemical classes outside small molecules, lipids, and peptides, or the instrument is not a TWIM-MS platform

## Inputs

- Raw calibration solution data as .mzML files (e.g. Waters Major Mix IMS/Tof Calibration Kit, Agilent ESI-L Tune Mix)
- Calibrant reference DTCCSN2 values from the Unified CCS Compendium (paired by calibrant identity)
- Instrument-specific EDC delay coefficient
- Charge-state annotations for peptide calibrants (optional, for multiply charged fits)

## Outputs

- One machine-readable calibration-parameter file per biomolecular class (small molecule, lipid, peptide) holding optimized power-law regression parameters
- Calibrant Gaussian apex arrival times extracted via DEIMoS

## How to apply

Load raw calibration data as .mzML files (e.g. Waters Major Mix IMS/Tof Calibration Kit, Agilent ESI-L Tune Mix, poly-dl-alanine peptides) into MOCCal's RawDT workflow, and supply the instrument-specific EDC delay coefficient. Extract calibrant arrival-time distributions with DEIMoS v1.3.2, fit each to a Gaussian distribution, and record the Gaussian apex as the feature's arrival time. Pair each arrival time with the calibrant's reference DTCCSN2 value from the Unified CCS Compendium, then optimize class-specific power-law regression parameters with nonlinear least squares — separately for the lipid class, the small-molecule class, and the peptide class (singly charged; optionally multiply charged peptide calibrants if charge-state information is present). Write each fitted parameter set to a machine-readable calibration-parameter file, one per biomolecular class, since each class has distinct fit parameters for the power law calibration curve. Alternatively, if processed arrival times already exist, use the UserDT executable path, which requires no Python installation but does require an 'Output' folder alongside the MOCCal executable.

## Related tools

- **MOCCal** (Python application that loads calibrant data, optimizes class-specific power-law regression parameters, and writes the calibration-parameter files) — https://github.com/HinesLab/MOCCal
- **DEIMoS** (Used (version 1.3.2) to extract calibrant arrival-time data from .mzML files and fit each arrival-time distribution to a Gaussian whose apex becomes the arrival time) — http://github.com/pnnl/deimos

## Examples

```
python MOCCal_RawDT.py  # run inside the DEIMoS v1.3.2 virtual environment with .mzML calibrant files and Unified CCS Compendium DTCCSN2 references
```

## Evaluation signals

- The fitted lipid power law must reproduce the reported lipid example feature (m/z 692.517) calibrated TWCCSN2 within 0.18% of its DTCCSN2 reference value
- The same m/z 692.517 feature calibrated with small-molecule and peptide power laws should show much larger deviations (1.8% and 2.6%, offsets of 38.7 and 25.3 Å2 from trendlines), confirming class specificity
- Fitted parameters reproduce the paper's error criterion of ±0.5–1% for correctly class-assigned calibrants
- One calibration-parameter file is emitted per biomolecular class, and running the UserDT path on the same calibrants gives consistent parameters

## Limitations

- Parameters are class-specific: applying a single power-law calibration across classes introduces calibration biases (1.8–2.6% error for the m/z 692.517 lipid example)
- MOCCal is currently scoped to small molecule, lipid, and peptide calibrations on TWIM-MS platforms
- Negative ionization mode data sets and other endogenous or xenobiotic chemical classes are not yet supported; user-driven expansion is welcomed
- The RawDT path requires installing pnnl's DEIMoS v1.3.2 in a virtual environment; the UserDT executable requires a folder titled 'Output' adjacent to it

## Evidence

- [results] arrival time data for the calibrants are extracted using the open-source software DEIMoS and fit to a Gaussian distribution. The apex of the Gaussian distribution is recorded as the feature's arrival: [evidence span withheld — anchor status: not_found]
- [results] Parameters are optimized for class-specific power law regressions using the calibrant's arrival time and reference drift tube CCS value: [evidence span withheld — anchor status: not_found]
- [readme] If you plan to use raw calibration files, you will need to install pnnl's DEIMoS, version 1.3.2 (http://github.com/pnnl/deimos). After DEIMoS is set up, you can then run MOCCal_RawDT.py: "If you plan to use raw calibration files, you will need to install pnnl's DEIMoS, version 1.3.2 (http://github.com/pnnl/deimos). After DEIMoS is set up, you can then run MOCCal_RawDT.py"
- [results] regression lines developed from DTCCSN2 values in the Unified CCS Compendium: "regression lines developed from DTCCSN2 values in the Unified CCS Compendium (Supporting Information Document 2, Tables 1–3)"
- [results] each biomolecular class has distinct fit parameters for the power law calibration curve: "each biomolecular class has distinct fit parameters for the power law calibration curve"
