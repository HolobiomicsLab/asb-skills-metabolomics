---
name: twim-ccs-power-law-calibration
description: Use when processing traveling-wave IM-MS data (e.g. Waters SYNAPT XS)
  where feature CCS values must be calibrated without prior identification, especially
  in mixed multiomic samples spanning lipids, small molecules, and peptides.
when_to_use_negative:
- Data were acquired in negative ionization mode — MOCCal is not yet extended to negative-mode
  datasets per the authors
- Analytes fall outside the small molecule, lipid, and peptide classes the tool was
  built around (e.g. other endogenous/xenobiotic chemical classes)
- The input is already identified features where structurally matched CCS values are
  known — class assignment without prior identification is unnecessary
edam_operation: http://edamontology.org/operation_0322
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: MOCCal
  role: Python application that fits class-specific power-law CCS calibrations, assigns
    unknown features to biomolecular classes, and computes class-specific TWCCSN2
    values
  repo: https://github.com/HinesLab/MOCCal
- name: DEIMoS
  role: Extracts calibrant arrival-time distributions from raw .mzML files and fits
    them to Gaussian distributions (apex = arrival time); required v1.3.2 for the
    RawDT workflow
  repo: http://github.com/pnnl/deimos
- name: Waters SYNAPT XS
  role: Traveling wave ion mobility mass spectrometer on which calibrant and experimental
    data were collected
provenance:
  source_task_ids:
  - task_004
  - task_001
  source_papers:
  - doi: 10.1021/acs.analchem.3c04290
    title: 'MOCCal: A Multiomic CCS Calibrator for Traveling Wave Ion Mobility Mass
      Spectrometry'
  source_content_hash: sha256:6b7d4f79bd7fc6f2dd027b7cbdc8f7706a21ddd708160fe0b9aa0b1e4d98c672
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/HinesLab/MOCCal
  tools:
  - CCSBase
  - DEIMoS
  - MOCCal
  - Waters AQUITY UPLC BEH amide
  - Waters SYNAPT XS
  edam_operation: http://edamontology.org/operation_0322
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# twim-ccs-power-law-calibration

## Summary

Fits class-specific (small molecule, lipid, peptide) power-law regression equations relating TWIM arrival times to reference drift-tube CCS (DTCCSN2) values, then calibrates unknown feature arrival times to accurate TWCCSN2 values. Implemented in MOCCal with DEIMoS for arrival-time extraction from raw .mzML calibration files.

## When to use

Use when processing traveling-wave IM-MS data (e.g. Waters SYNAPT XS) where feature CCS values must be calibrated without prior identification, especially in mixed multiomic samples spanning lipids, small molecules, and peptides.

## When NOT to use

- Data were acquired in negative ionization mode — MOCCal is not yet extended to negative-mode datasets per the authors
- Analytes fall outside the small molecule, lipid, and peptide classes the tool was built around (e.g. other endogenous/xenobiotic chemical classes)
- The input is already identified features where structurally matched CCS values are known — class assignment without prior identification is unnecessary

## Inputs

- Raw calibration .mzML files from TWIM-MS runs of calibrant mixtures (Waters Major Mix IMS/Tof Calibration Kit, SPLASH Lipidomix, Agilent ESI-L Tune Mix)
- Reference DTCCSN2 values from the Unified CCS Compendium paired with calibrant arrival times (UserDT/RawDT templates)
- Instrument-specific EDC delay coefficient
- Experimental feature table (spreadsheet: feature identifier, m/z, charge state, retention time, arrival time)

## Outputs

- Per-class power-law calibration parameter sets written to machine-readable calibration-parameter files (one per biomolecular class)
- Calibrated TWCCSN2 value per feature from the selected class-specific calibration
- Assigned biomolecular class per feature with calibration-effect score (1-4 based on percent difference to next closest match; 0 for multiply charged)

## How to apply

Load raw calibrant data as .mzML files (e.g. Waters Major Mix IMS/Tof Calibration Kit, SPLASH Lipidomix, Agilent ESI-L Tune Mix, MassPREP enolase/ADH digests, poly-dl-alanine) and extract arrival-time distributions with DEIMoS v1.3.2, fitting each to a Gaussian and recording the apex as the arrival time. For each biomolecular class, optimize power-law regression parameters against the calibrants' reference DTCCSN2 values from the Unified CCS Compendium using the user-supplied instrument-specific EDC delay coefficient, and write each parameter set to a machine-readable calibration file. For unknown features (spreadsheet of feature ID, m/z, charge, retention time, arrival time), compute up to three calibrated TWCCSN2 values, then select the calibration whose calibrated value has the smallest ΔCCS to the corresponding DTCCSN2-vs-m/z trendline, respecting m/z constraints (small molecules < m/z 550, lipids > m/z 300; fallback to next closest match when outside constraints). Class rules are small molecule m/z < 550, lipid m/z > 300, peptide charge > 1; each class has distinct power-law fit parameters, which is the rationale for class-specific calibration over a single equation.

## Related tools

- **MOCCal** (Python application that fits class-specific power-law CCS calibrations, assigns unknown features to biomolecular classes, and computes class-specific TWCCSN2 values) — https://github.com/HinesLab/MOCCal
- **DEIMoS** (Extracts calibrant arrival-time distributions from raw .mzML files and fits them to Gaussian distributions (apex = arrival time); required v1.3.2 for the RawDT workflow) — http://github.com/pnnl/deimos
- **Waters SYNAPT XS** (Traveling wave ion mobility mass spectrometer on which calibrant and experimental data were collected)

## Examples

```
python MOCCal_RawDT.py  # run in the DEIMoS v1.3.2 virtual environment with raw calibrant .mzML files and an 'Output' folder present
```

## Evaluation signals

- Fitted lipid power law should reproduce the example lipid feature at m/z 692.517 within 0.18% of its DTCCSN2 reference value (versus 1.8% and 2.6% error for small-molecule and peptide power laws)
- Per-class fit RMSE should approximate ~0.28 Å2 (lipid), ~0.58 Å2 (small molecule), ~0.12 Å2 (peptide)
- On NIST SRM 1950 single-phase extract, ≥88.6% of 44 lipid features and ≥93.3% of 15 small-molecule features should be assigned to their structurally matched calibrant class
- Class assignment rules must be honored: small molecule m/z < 550, lipid m/z > 300, peptide charge > 1, with m/z-constraint fallback to next closest ΔCCS match
- Calibration-effect scores fall in the 1-4 range (score 1 = <1% difference to next closest match, score 4 = ≥6%)

## Limitations

- MOCCal is limited to small molecule, lipid, and peptide calibrations within TWIM-MS constraints
- Not yet extended to negative ionization mode data sets or other endogenous/xenobiotic chemical classes
- A single calibration equation can introduce calibration biases that confound feature identification; class-specific fitting is required
- Terms 'arrival time' and 'drift time' are used interchangeably in the software, though TWIM records arrival time rather than drift time within the cell

## Evidence

- [abstract] Here we describe the development of a multiomic CCS calibration tool, MOCCal, to automate the assignment of unknown features to the power law calibration that provides the most accurate CCS value.: "Here we describe the development of a multiomic CCS calibration tool, MOCCal, to automate the assignment of unknown features to the power law calibration that provides the most accurate CCS value."
- [results] arrival time data for the calibrants are extracted using the open-source software DEIMoS and fit to a Gaussian distribution. The apex of the Gaussian distribution is recorded as the feature's arrival: [evidence span withheld — anchor status: not_found]
- [results] less than m/z 550 for small molecules; greater than m/z 300 for lipids. If the feature falls outside the m/z constraints for the assigned class, the next closest match by ΔCCS will be selected: [evidence span withheld — anchor status: not_found]
- [results] the calibrated TWCCSN2 value from the lipid-fit power law is 0.18% from the DTCCSN2 value whereas the small molecule and peptide power laws yield calibrated TWCCSN2 values with 1.8 and 2.6% error: "the calibrated TWCCSN2 value from the lipid-fit power law is 0.18% from the DTCCSN2 value whereas the small molecule and peptide power laws yield calibrated TWCCSN2 values with 1.8 and 2.6% error"
- [readme] If you plan to use raw calibration files, you will need to install pnnl’s DEIMoS, version 1.3.2 (http://github.com/pnnl/deimos).: "If you plan to use raw calibration files, you will need to install pnnl’s DEIMoS, version 1.3.2 (http://github.com/pnnl/deimos)."
- [discussion] User-driven expansion of MOCCal for the calibration of negative ionization mode data sets and the incorporation of other endogenous or xenobiotic chemical classes is welcomed: "User-driven expansion of MOCCal for the calibration of negative ionization mode data sets and the incorporation of other endogenous or xenobiotic chemical classes is welcomed"
