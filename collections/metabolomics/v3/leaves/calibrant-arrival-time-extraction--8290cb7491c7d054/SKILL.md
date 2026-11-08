---
name: calibrant-arrival-time-extraction
description: Use when raw TWIM-MS calibration solution data (.mzML, e.g. Waters Major
  Mix or Agilent Tune Mix) must be converted into arrival-time inputs paired with
  reference DTCCSN2 values to build class-specific CCS power-law calibrations.
when_to_use_negative:
- When calibration data are already processed — use the UserDT executable path of
  MOCCal instead, which needs no Python/DEIMoS installation and takes user-supplied
  drift/arrival-time tables
- When the analyte class is outside small molecules, lipids, or peptides, or the data
  are negative ionization mode — MOCCal's calibrations are currently limited to these
  classes on TWIM-MS platforms
- When the CCS platform is drift-tube or TIMS rather than traveling-wave IM — the
  arrival time recorded is the detector time on TWIM platforms, not drift time, so
  the power-law fit is TWIM-specific
tools:
- name: MOCCal
  role: Python application whose RawDT workflow consumes raw .mzML calibration files,
    orchestrates DEIMoS extraction, and fits the class-specific power-law calibration
    parameters
  repo: https://github.com/HinesLab/MOCCal
- name: DEIMoS
  role: Open-source tool used inside the RawDT path to extract calibrant arrival-time
    data from .mzML and fit each feature to a Gaussian distribution; version 1.3.2
    required for the paper's workflow
  repo: http://github.com/pnnl/deimos
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/acs.analchem.3c04290
    title: 'MOCCal: A Multiomic CCS Calibrator for Traveling Wave Ion Mobility Mass
      Spectrometry'
  source_content_hash: sha256:0bf73301077bc8a927838a192c115575037aa8a1c8fdb5e68cb452ad9df057b9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/HinesLab/MOCCal
  tools:
  - DEIMoS
  - MOCCal
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# calibrant-arrival-time-extraction

## Summary

Extracts calibrant arrival times from raw TWIM-MS calibration files (.mzML) using DEIMoS Gaussian-apex extraction, then optimizes class-specific power-law regression parameters for CCS calibration of small molecules, lipids, and peptides in the MOCCal workflow.

## When to use

Use when raw TWIM-MS calibration solution data (.mzML, e.g. Waters Major Mix or Agilent Tune Mix) must be converted into arrival-time inputs paired with reference DTCCSN2 values to build class-specific CCS power-law calibrations.

## When NOT to use

- When calibration data are already processed — use the UserDT executable path of MOCCal instead, which needs no Python/DEIMoS installation and takes user-supplied drift/arrival-time tables
- When the analyte class is outside small molecules, lipids, or peptides, or the data are negative ionization mode — MOCCal's calibrations are currently limited to these classes on TWIM-MS platforms
- When the CCS platform is drift-tube or TIMS rather than traveling-wave IM — the arrival time recorded is the detector time on TWIM platforms, not drift time, so the power-law fit is TWIM-specific

## Inputs

- Raw calibration data for each calibration solution as .mzML files (collected on a Waters SYNAPT XS TWIM-MS instrument)
- Reference DTCCSN2 values for calibrants from the Unified CCS Compendium (e.g. Waters Major Mix IMS/Tof Calibration Kit, Agilent ESI-L Tune Mix, MassPREP enolase digest)
- Instrument-specific EDC delay coefficient supplied by the user

## Outputs

- Per-calibrant arrival times (Gaussian-fit apexes extracted by DEIMoS)
- Fitted class-specific power-law calibration parameter sets, one calibration-parameter file per biomolecular class (small molecule, lipid, peptide)

## How to apply

Load the raw calibration data for each calibration solution as .mzML files into the RawDT path of MOCCal, running MOCCal_RawDT.py inside pnnl's DEIMoS virtual environment (version 1.3.2 is specified for the paper). Use DEIMoS to extract arrival-time data for each calibrant and fit the extracted ion signal to a Gaussian distribution, recording the Gaussian apex as the feature's arrival time. Pair each calibrant arrival time with its reference drift-tube CCS value (DTCCSN2) from the Unified CCS Compendium, and supply the instrument-specific EDC delay coefficient. Optimize parameters for class-specific power-law regressions by fitting arrival time against DTCCSN2 separately for the lipid class, the small-molecule class, and the peptide class (singly charged, optionally multiply charged if charge-state information is present), then write each fitted parameter set to a machine-readable calibration-parameter file per biomolecular class.

## Related tools

- **MOCCal** (Python application whose RawDT workflow consumes raw .mzML calibration files, orchestrates DEIMoS extraction, and fits the class-specific power-law calibration parameters) — https://github.com/HinesLab/MOCCal
- **DEIMoS** (Open-source tool used inside the RawDT path to extract calibrant arrival-time data from .mzML and fit each feature to a Gaussian distribution; version 1.3.2 required for the paper's workflow) — http://github.com/pnnl/deimos

## Examples

```
python MOCCal_RawDT.py  # run inside the DEIMoS 1.3.2 virtual environment, with raw .mzML calibration files per the RawDT data templates
```

## Evaluation signals

- The fitted lipid-class power law reproduces the reported lipid example feature (m/z 692.517) with a calibrated TWCCSN2 value within 0.18% of its DTCCSN2 reference value, consistent with the paper's ±0.5–1% error criterion for correct class assignment
- Each calibrant's extracted arrival time equals the apex of a fitted Gaussian distribution (not a centroid or maximum bin)
- One machine-readable calibration-parameter file exists per biomolecular class, each with distinct fit parameters (the paper notes each class has distinct power-law fit parameters)
- Correct-class calibrants calibrate with ≤1–3% deviation; the paper's effect-score scheme treats <1% as score 1 and ≥6% as score 4, so grossly mismatched class calibration parameters should be flagged

## Limitations

- Raw-data processing requires specifically pnnl's DEIMoS, version 1.3.2, installed in a dedicated virtual environment; the UserDT executable path cannot accept raw .mzML
- TWIM platforms record arrival time (time at detector), not drift time within the TWIM cell — the two terms are used interchangeably in the software but are physically distinct
- MOCCal is currently limited to small-molecule, lipid, and peptide calibrations on TWIM-MS platforms and is not yet extended to negative ionization mode data sets or other endogenous/xenobiotic chemical classes

## Evidence

- [results] raw data for each calibration solution should be loaded as a .mzML file: "raw data for each calibration solution should be loaded as a .mzML file"
- [results] arrival time data for the calibrants are extracted using the open-source software DEIMoS and fit to a Gaussian distribution. The apex of the Gaussian distribution is recorded as the feature's arrival: [evidence span withheld — anchor status: not_found]
- [results] Parameters are optimized for class-specific power law regressions using the calibrant's arrival time and reference drift tube CCS value: [evidence span withheld — anchor status: not_found]
- [readme] install pnnl's DEIMoS, version 1.3.2 (http://github.com/pnnl/deimos): "you will need to install pnnl's DEIMoS, version 1.3.2 (http://github.com/pnnl/deimos)"
- [results] MOCCal utilizes biomolecule-specific TWIM CCS calibration equations: "MOCCal utilizes biomolecule-specific TWIM CCS calibration equations"
