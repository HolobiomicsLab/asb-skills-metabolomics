---
name: calibration-parameter-file-export
description: Use when class-specific power-law calibration parameters have been fitted
  from calibrant arrival times and reference DTCCSN2 values and must be persisted
  as machine-readable files that MOCCal can consume for class assignment.
when_to_use_negative:
- The experimental feature table only needs existing calibration parameters applied
  (class assignment / TWCCSN2 calculation), not fitting and export of new ones
- Data were collected on drift-tube or TIMS instruments rather than a traveling-wave
  IM-MS platform such as the Waters SYNAPT XS, where the TWIM power-law form does
  not apply
- Calibrants fall outside the three supported biomolecular classes (small molecules,
  lipids, peptides), e.g., other endogenous or xenobiotic chemical classes
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: MOCCal
  role: Python application that fits the class-specific power-law parameters from
    calibrant arrival times and writes each fitted parameter set to a machine-readable
    calibration-parameter file
  repo: https://github.com/HinesLab/MOCCal
- name: DEIMoS
  role: Used in the RawDT path to extract calibrant arrival times from .mzML raw files
    via Gaussian peak fitting before parameter optimization
  repo: http://github.com/pnnl/deimos
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/acs.analchem.3c04290
    title: 'MOCCal: A Multiomic CCS Calibrator for Traveling Wave Ion Mobility Mass
      Spectrometry'
  source_content_hash: sha256:44914bef770251e84c8cc0e35a3370da770a9b42692bf80eb1a25cdc9c40e1fa
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/HinesLab/MOCCal
  tools:
  - DEIMoS
  - MOCCal
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 5
---

# calibration-parameter-file-export

## Summary

Export the fitted class-specific traveling-wave power-law calibration parameters (for lipids, small molecules, and peptides) from MOCCal's calibration workflow into machine-readable parameter files, one per biomolecular class, so downstream class assignment and TWCCSN2 calculation can apply them.

## When to use

Use when class-specific power-law calibration parameters have been fitted from calibrant arrival times and reference DTCCSN2 values and must be persisted as machine-readable files that MOCCal can consume for class assignment.

## When NOT to use

- The experimental feature table only needs existing calibration parameters applied (class assignment / TWCCSN2 calculation), not fitting and export of new ones
- Data were collected on drift-tube or TIMS instruments rather than a traveling-wave IM-MS platform such as the Waters SYNAPT XS, where the TWIM power-law form does not apply
- Calibrants fall outside the three supported biomolecular classes (small molecules, lipids, peptides), e.g., other endogenous or xenobiotic chemical classes

## Inputs

- Raw calibrant arrival-time data as .mzML files (e.g., Waters Major Mix IMS/Tof Calibration Kit or Agilent ESI-L Tune Mix runs) or processed calibrant arrival-time templates from the UserDT folder
- Reference DTCCSN2 values from the Unified CCS Compendium paired with calibrant arrival times
- Instrument-specific EDC delay coefficient supplied by the user

## Outputs

- One machine-readable calibration-parameter file per biomolecular class (lipid, small molecule, peptide), each containing the fitted power-law regression parameters

## How to apply

First fit the class-specific power-law regressions: for each biomolecular class (lipids, small molecules, peptides), load the calibrant raw data as .mzML files (RawDT path) or processed arrival-time templates (UserDT path), extract arrival times with DEIMoS by fitting each calibrant peak to a Gaussian distribution and recording the apex, then optimize the power-law parameters against the calibrant's reference DTCCSN2 value from the Unified CCS Compendium, supplying the instrument-specific EDC delay coefficient. Write each fitted parameter set to its own machine-readable calibration-parameter file, one per biomolecular class, so MOCCal can later calibrate each experimental arrival time with up to three class-specific calibration equations. Validate the lipid-class parameter file by confirming the lipid example feature at m/z 692.517 yields a calibrated TWCCSN2 value within 0.18% of its DTCCSN2 reference, consistent with the paper's ±0.5–1% error criterion for correct class assignment.

## Related tools

- **MOCCal** (Python application that fits the class-specific power-law parameters from calibrant arrival times and writes each fitted parameter set to a machine-readable calibration-parameter file) — https://github.com/HinesLab/MOCCal
- **DEIMoS** (Used in the RawDT path to extract calibrant arrival times from .mzML raw files via Gaussian peak fitting before parameter optimization) — http://github.com/pnnl/deimos

## Examples

```
python MOCCal_RawDT.py  # run in the DEIMoS 1.3.2 virtual environment with .mzML calibrant files to fit and export per-class parameter files
```

## Evaluation signals

- Exactly one parameter file is produced per biomolecular class (lipid, small molecule, peptide), each machine-readable
- The fitted lipid-class power law reproduces the reported lipid example feature (m/z 692.517) calibrated TWCCSN2 within 0.18% of its DTCCSN2 reference value
- Mispplied-class parameter files give the documented larger errors (1.8% and 2.6% for the m/z 692.517 lipid via small-molecule and peptide power laws), confirming parameters are class-specific and distinct
- The parameter files, when used for class assignment, achieve ≥70% correct assignments for lipids and peptides with calibration errors within ±1%
- Fit residuals against reference DTCCSN2 values fall within the paper's ±0.5–1% criterion for correctly assigned classes

## Limitations

- Raw-data (RawDT) parameter fitting requires pnnl's DEIMoS, version 1.3.2, installed in the Python environment; the UserDT executable path requires no Python but the executable must sit next to a folder titled 'Output'
- MOCCal is scoped to small molecule, lipid, and peptide calibrations on TWIM-MS platforms and has not been extended to negative ionization mode data sets or other chemical classes
- Arrival time and drift time are used interchangeably in the software, but TWIM records arrival time at the detector, not drift time within the cell — users must not substitute drift-tube semantics when fitting parameters

## Evidence

- [results] Optimize parameters for class-specific power law regressions using calibrant arrival time and reference drift tube CCS value: [evidence span withheld — anchor status: not_found]
- [results] raw calibrant data must be .mzML: "raw data for each calibration solution should be loaded as a .mzML file"
- [results] DEIMoS Gaussian apex extraction of arrival times: [evidence span withheld — anchor status: not_found]
- [readme] RawDT path requires DEIMoS 1.3.2: "If you plan to use raw calibration files, you will need to install pnnl’s DEIMoS, version 1.3.2 (http://github.com/pnnl/deimos)."
- [results] Each class has distinct fit parameters: "each biomolecular class has distinct fit parameters for the power law calibration curve"
