---
name: class-specific-ccs-parameter-optimization
description: Use when raw or processed TWIM-MS calibrant data (e.g. Waters Major Mix,
  Agilent ESI-L Tune Mix) must be converted into class-specific power-law calibration
  parameter sets for TWCCSN2 calculation.
when_to_use_negative:
- Data were acquired in negative ionization mode — MOCCal has not yet been extended
  to negative-mode data sets
- Analytes fall outside the small molecule, lipid, and peptide classes supported by
  MOCCal's calibrations
- A single generalized power-law calibration is sufficient (e.g. single-class analyses
  where class assignment biases do not apply)
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: MOCCal
  role: Python application that loads calibration data, optimizes class-specific power-law
    parameters, and writes calibration-parameter files
  repo: https://github.com/HinesLab/MOCCal
- name: DEIMoS
  role: Extracts calibrant arrival-time data from .mzML raw files and fits each distribution
    to a Gaussian; the apex is recorded as the arrival time (v1.3.2 required for RawDT
    path)
  repo: http://github.com/pnnl/deimos
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/acs.analchem.3c04290
    title: 'MOCCal: A Multiomic CCS Calibrator for Traveling Wave Ion Mobility Mass
      Spectrometry'
  source_content_hash: sha256:7f373392e7cf582a6d9887baf4e045799805fc2f25587b264cd2ab1265d61852
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
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# class-specific-ccs-parameter-optimization

## Summary

Fits separate power-law CCS calibration regressions for small molecules, lipids, and peptides by optimizing each class's parameters against calibrant arrival times and reference DTCCSN2 values from the Unified CCS Compendium, enabling accurate TWCCSN2 determination on Waters TWIM-MS instruments.

## When to use

Use when raw or processed TWIM-MS calibrant data (e.g. Waters Major Mix, Agilent ESI-L Tune Mix) must be converted into class-specific power-law calibration parameter sets for TWCCSN2 calculation.

## When NOT to use

- Data were acquired in negative ionization mode — MOCCal has not yet been extended to negative-mode data sets
- Analytes fall outside the small molecule, lipid, and peptide classes supported by MOCCal's calibrations
- A single generalized power-law calibration is sufficient (e.g. single-class analyses where class assignment biases do not apply)

## Inputs

- Raw calibrant .mzML files (e.g. Waters Major Mix IMS/Tof Calibration Kit, Agilent ESI-L Tune Mix) for the RawDT path
- Processed calibrant arrival times via the MOCCal UserDT data templates
- Reference DTCCSN2 values from the Unified CCS Compendium (Supporting Information Tables 1-3)
- Instrument-specific EDC delay coefficient

## Outputs

- Machine-readable calibration-parameter files, one per biomolecular class (small molecules, lipids, peptides)
- Fitted power-law regression parameters for class-specific TWCCSN2 calculation

## How to apply

Load calibrant raw data (.mzML) or processed calibration spreadsheets from the MOCCal RawDT/UserDT templates, pairing calibrant arrival times with reference DTCCSN2 values from the Unified CCS Compendium. Extract arrival times with DEIMoS (v1.3.2), fitting each calibrant's arrival-time distribution to a Gaussian and recording the apex. Supply the instrument-specific EDC delay coefficient, then optimize parameters for three separate power-law regressions via nonlinear least-squares: one for lipids, one for small molecules, and one for peptides (singly charged calibrants, optionally multiply charged if charge-state information is present). Write each fitted parameter set to a machine-readable calibration-parameter file, one per biomolecular class. Distinct class-specific parameters matter because each biomolecular class has different power-law fit parameters on the same TWIM instrument (Waters SYNAPT XS).

## Related tools

- **MOCCal** (Python application that loads calibration data, optimizes class-specific power-law parameters, and writes calibration-parameter files) — https://github.com/HinesLab/MOCCal
- **DEIMoS** (Extracts calibrant arrival-time data from .mzML raw files and fits each distribution to a Gaussian; the apex is recorded as the arrival time (v1.3.2 required for RawDT path)) — http://github.com/pnnl/deimos

## Examples

```
python MOCCal_RawDT.py  # run in the DEIMoS v1.3.2 virtual environment with calibrant .mzML files in the RawData folder
```

## Evaluation signals

- The fitted lipid power law should reproduce the example lipid feature (m/z 692.517) TWCCSN2 within 0.18% of its DTCCSN2 reference value, consistent with the paper's ±0.5-1% accuracy criterion for correct class assignment
- Wrong-class calibrations of the same feature should show much larger errors (small molecule: 1.8%, peptide: 2.6%), confirming the class-specific fits are distinct
- Three separate parameter files (small molecule, lipid, peptide) are produced, one per biomolecular class
- Gaussian fits to calibrant arrival-time distributions should yield well-defined apexes recorded as arrival times
- Fits run to completion in the DEIMoS v1.3.2 virtual environment (RawDT) or the MOCCal executable with an 'Output' folder present (UserDT)

## Limitations

- MOCCal calibrations are currently limited to small molecule, lipid, and peptide classes within TWIM-MS platform constraints
- Not yet extended to negative ionization mode data sets or other endogenous/xenobiotic chemical classes
- Users must supply the correct instrument-specific EDC delay coefficient; arrival time and drift time are used interchangeably but TWIM records detector arrival time, not drift time within the cell

## Evidence

- [results] Optimize parameters for class-specific power law regressions using calibrant arrival time and reference drift tube CCS value: [evidence span withheld — anchor status: not_found]
- [results] DEIMoS Gaussian apex extraction: [evidence span withheld — anchor status: not_found]
- [readme] RawDT installation requirement: "If you plan to use raw calibration files, you will need to install pnnl's DEIMoS, version 1.3.2 (http://github.com/pnnl/deimos). After DEIMoS is set up, you can then run MOCCal_RawDT.py"
- [results] Distinct class-specific fit parameters: "each biomolecular class has distinct fit parameters for the power law calibration curve"
- [discussion] Scope limitation: "MOCCal around small molecule, lipid, and peptide calibrations with the intention of enabling mixed multiomic analyses on TWIM-MS platforms"
