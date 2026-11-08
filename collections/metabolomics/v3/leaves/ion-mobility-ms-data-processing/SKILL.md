---
name: ion-mobility-ms-data-processing
description: Use when analyzing high-dimensional TWIM-MS data from mixed multiomic
  samples (small molecules, lipids, peptides) on a Waters SYNAPT XS where calibrated
  CCS values and biomolecular class assignments are needed for features without prior
  identification, as when a feature table derived from a NIST.
when_to_use_negative:
- Input data is not traveling-wave ion mobility (e.g., drift tube IMS or TIMS) — MOCCal
  is built around TWIM CCS calibration equations
- Calibration is needed for chemical classes other than small molecules, lipids, or
  peptides (e.g., negative ionization mode datasets or xenobiotic classes) — MOCCal
  does not yet support them
- The experimental dataset contains only multiply charged species with no singly charged
  features — MOCCal's class assignment workflow is built around singly charged features
  with optional multiply charged calibration scores defaulted to 0
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: MOCCal
  role: 'Core tool: performs CCS calibration parameter determination, biomolecular
    class assignment, and class-specific TWCCSN2 calibration of experimental features
    from TWIM-MS data.'
  repo: https://github.com/HinesLab/MOCCal
- name: DEIMoS
  role: Used in the RawDT workflow to extract calibrant arrival times from raw .mzML
    files and fit them to Gaussian distributions, feeding the apex arrival times into
    MOCCal's power law regressions.
  repo: http://github.com/pnnl/deimos
- name: Waters SYNAPT XS
  role: Instrument used to acquire the traveling-wave ion mobility mass spectrometry
    data processed by this skill.
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1021/acs.analchem.3c04290
    title: 'MOCCal: A Multiomic CCS Calibrator for Traveling Wave Ion Mobility Mass
      Spectrometry'
  source_content_hash: sha256:fa05ab04a98c694bb6b67c401f2c305b719336aa546996edf9aeb91c7c7d10b5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/HinesLab/MOCCal
  tools:
  - Agilent ESI-L Tune Mix
  - DEIMoS
  - MOCCal
  - NIST SRM 1950
  - Waters SYNAPT XS
  - poly-dl-alanine
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 17
---

# ion-mobility-ms-data-processing

## Summary

Skill for processing traveling-wave ion mobility mass spectrometry (TWIM-MS) data with MOCCal, which assigns features to biomolecular classes and computes class-specific calibrated TWCCSN2 values using structurally matched power law CCS calibrations without prior feature identification.

## When to use

Use when analyzing high-dimensional TWIM-MS data from mixed multiomic samples (small molecules, lipids, peptides) on a Waters SYNAPT XS where calibrated CCS values and biomolecular class assignments are needed for features without prior identification, as when a feature table derived from a NIST SRM 1950 plasma extract spiked with protein digests must be class-assigned and CCS-calibrated.

## When NOT to use

- Input data is not traveling-wave ion mobility (e.g., drift tube IMS or TIMS) — MOCCal is built around TWIM CCS calibration equations
- Calibration is needed for chemical classes other than small molecules, lipids, or peptides (e.g., negative ionization mode datasets or xenobiotic classes) — MOCCal does not yet support them
- The experimental dataset contains only multiply charged species with no singly charged features — MOCCal's class assignment workflow is built around singly charged features with optional multiply charged calibration scores defaulted to 0

## Inputs

- Raw TWIM-MS calibration data as .mzML files (e.g., Waters Major Mix IMS/Tof Calibration Kit or Agilent ESI-L Tune Mix)
- Processed calibration spreadsheets (calibrant arrival times and reference DTCCSN2 values) for the UserDT version
- Experimental feature table spreadsheet with feature identifiers, m/z, charge state, retention time, and arrival time values

## Outputs

- Calibrated TWCCSN2 values per feature from up to three class-specific power law equations
- Biomolecular class assignment (small molecule, lipid, peptide) per feature based on smallest ΔCCS to trendline
- Class assignment effect score (1–4, with 4 = ≥6% difference; 0 default for multiply charged assignments)
- Output folder containing per-class calibration error percentages and RMSE values

## How to apply

Load raw calibration data (e.g., Waters Major Mix IMS/Tof Calibration Kit or Agilent ESI-L Tune Mix) as .mzML files into MOCCal; if using raw data, install pnnl's DEIMoS version 1.3.2 and run MOCCal_RawDT.py so DEIMoS extracts calibrant arrival times and fits them to a Gaussian distribution, recording the apex. Optimize parameters for class-specific power law regressions using each calibrant's arrival time and reference drift tube CCS (DTCCSN2) value from the Unified CCS Compendium. Import the experimental feature table (feature identifiers, m/z, charge state, retention time, arrival time) and let MOCCal calculate up to three calibrated TWCCSN2 values per singly charged feature — one each from the small molecule, lipid, and peptide (z = 1, 2, 3) power law equations. Select the calibration whose calibrated TWCCSN2 value has the smallest ΔCCS to the corresponding trendline, applying m/z constraints (<550 for small molecules, >300 for lipids); if a feature falls outside the m/z constraints for the assigned class, take the next closest match by ΔCCS. The per-class calibration errors for the NIST SRM 1950 flow-injection dataset were 0.02% (lipids), 0.75% (small molecules), and 0.01%, 0.06%, and 0.13% for peptides at z = 1, 2, 3, with RMSE values of 0.28, 0.56, 0.09, 0.09, and 0.24 respectively.

## Related tools

- **MOCCal** (Core tool: performs CCS calibration parameter determination, biomolecular class assignment, and class-specific TWCCSN2 calibration of experimental features from TWIM-MS data.) — https://github.com/HinesLab/MOCCal
- **DEIMoS** (Used in the RawDT workflow to extract calibrant arrival times from raw .mzML files and fit them to Gaussian distributions, feeding the apex arrival times into MOCCal's power law regressions.) — http://github.com/pnnl/deimos
- **Waters SYNAPT XS** (Instrument used to acquire the traveling-wave ion mobility mass spectrometry data processed by this skill.)

## Examples

```
python MOCCal_RawDT.py  # after installing pnnl's DEIMoS, version 1.3.2, in the DEIMoS virtual environment; alternatively, run the UserDT executable with an 'Output' folder in the same directory
```

## Evaluation signals

- Per-class calibration errors for the flow-injection NIST SRM 1950 dataset should reproduce the published values: 0.02% (lipids), 0.75% (small molecules), and 0.01%, 0.06%, and 0.13% for peptides at z = 1, 2, 3
- Per-class RMSE values should match the published 0.28, 0.56, 0.09, 0.09, and 0.24 for lipids, small molecules, and peptides z = 1, 2, 3
- A representative lipid (m/z 692.517) calibrated with the lipid power law should yield a TWCCSN2 value 0.18% from the DTCCSN2 reference, with the small molecule and peptide power laws yielding 1.8% and 2.6% error
- Small molecule features from the CCS Compendium should be assigned to the small molecule calibration 81–83% of the time with errors of ±0.5 to 5.0%
- MOCCal should achieve correct assignment for ≥70% of lipids and peptides when calibration errors are ±0.5%

## Limitations

- MOCCal is currently limited to small molecule, lipid, and peptide calibrations within TWIM-MS constraints and does not yet cover negative ionization mode datasets or other endogenous/xenobiotic chemical classes
- Calibration biases may confound feature identification when applying a single calibration equation to all classes
- Class assignment uses m/z constraints (<550 for small molecules, >300 for lipids); features outside the constraints for their assigned class will be reassigned to the next closest match by ΔCCS, which may reduce accuracy
- The terms arrival time and drift time are used interchangeably in the software, though TWIM platforms record arrival (drift-time-equivalent) rather than true drift time — outputs should be interpreted as TWCCSN2 values, not DTCCSN2

## Evidence

- [abstract] abstract: MOCCal description: "Here we describe the development of a multiomic CCS calibration tool, MOCCal, to automate the assignment of unknown features to the power law calibration that provides the most accurate CCS value."
- [results] results: arrival time extraction: [evidence span withheld — anchor status: not_found]
- [results] results: m/z constraints: [evidence span withheld — anchor status: not_found]
- [readme] readme: UserDT instructions: "download the MOCCal zip file ([MOCCalZip](https://drive.google.com/drive/folders/1emXyk9YWUuBdkcCdkMoD0RCHJLZxQZcr?usp=sharing)), extract all, and use the MOCCal executable file. The only requirement"
- [discussion] discussion: scope limitation: "MOCCal around small molecule, lipid, and peptide calibrations with the intention of enabling mixed multiomic analyses on TWIM-MS platforms"
