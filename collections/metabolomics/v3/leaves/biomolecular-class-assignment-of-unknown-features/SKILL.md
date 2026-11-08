---
name: biomolecular-class-assignment-of-unknown-features
description: Use when you have untargeted traveling-wave ion mobility data (feature
  m/z, charge state, retention time, arrival time) and must assign each feature to
  small molecule, lipid, or peptide class — and obtain a calibrated TWCCSN2 — without
  prior identification.
when_to_use_negative:
- When the calibration data set or ionization mode is outside MOCCal's current scope
  — it is parametrized for positive-ion-mode small molecule, lipid, and peptide calibrations
  and is not yet extended to negative ionization mode data or other endogenous/xenobiotic
  chemical classes
- When features are already structurally identified, since the skill exists specifically
  to assign unknown features to calibrations without prior identification
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_3520
tools:
- name: MOCCal
  role: Python application that calibrates each arrival time with up to three class-specific
    power law equations, applies the m/z class-assignment rules, and selects the calibration
    with the smallest ΔCCS
  repo: https://github.com/HinesLab/MOCCal
- name: DEIMoS
  role: Extracts calibrant arrival time distributions from .mzML raw files and fits
    each to a Gaussian; the apex is recorded as the feature's arrival time (version
    1.3.2 required for MOCCal RawDT mode)
  repo: http://github.com/pnnl/deimos
- name: Waters SYNAPT XS
  role: Traveling wave ion mobility mass spectrometer on which the TWIM-MS data to
    be class-assigned and calibrated are collected
provenance:
  source_task_ids:
  - task_004
  - task_007
  source_papers:
  - doi: 10.1021/acs.analchem.3c04290
    title: 'MOCCal: A Multiomic CCS Calibrator for Traveling Wave Ion Mobility Mass
      Spectrometry'
  source_content_hash: sha256:8dca23e80fbf0391d095cdc09a02dcb1a1c6e3c81f7d75794e75b7ee08bd0a5e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/HinesLab/MOCCal
  tools:
  - CCSBase
  - DEIMoS
  - MOCCal
  - Waters AQUITY UPLC BEH amide
  - Waters Acquity FTN I-Class Plus
  - Waters SYNAPT XS
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# biomolecular-class-assignment-of-unknown-features

## Summary

Assign unidentified TWIM-MS features to a biomolecular class (small molecule, lipid, or peptide) by calibrating each singly charged feature's arrival time with up to three class-specific power law equations and selecting the calibration whose TWCCSN2 value has the smallest ΔCCS to the DTCCSN2-vs-m/z regression trendline.

## When to use

Use when you have untargeted traveling-wave ion mobility data (feature m/z, charge state, retention time, arrival time) and must assign each feature to small molecule, lipid, or peptide class — and obtain a calibrated TWCCSN2 — without prior identification.

## When NOT to use

- When the calibration data set or ionization mode is outside MOCCal's current scope — it is parametrized for positive-ion-mode small molecule, lipid, and peptide calibrations and is not yet extended to negative ionization mode data or other endogenous/xenobiotic chemical classes
- When features are already structurally identified, since the skill exists specifically to assign unknown features to calibrations without prior identification

## Inputs

- Raw calibrant ion mobility data as .mzML files (e.g., Waters Major Mix IMS/Tof Calibration Kit, SPLASH Lipidomix, peptide digests)
- Experimental feature spreadsheet with feature identifiers, m/z, charge state, retention time, and arrival time values
- Reference DTCCSN2 values from the Unified CCS Compendium for power law regression fitting

## Outputs

- Per-feature biomolecular class assignment (small molecule, lipid, or peptide)
- Calibrated TWCCSN2 value per feature from the assigned class's power law
- Calibration-effect score (1–4, or 0 for multiply charged assignments)
- ΔCCS difference (Å²) between calibrated TWCCSN2 and each class's DTCCSN2-vs-m/z trendline

## How to apply

First build class-specific power law regressions from calibrant apex arrival times (Gaussian fits extracted in DEIMoS 1.3.2 from .mzML files of Waters Major Mix IMS/Tof Calibration Kit, SPLASH Lipidomix, and peptide calibrants) against reference DTCCSN2 values from the Unified CCS Compendium. Then calibrate each singly charged experimental feature with up to three class-specific power law equations and compute ΔCCS, the difference in Å² between the calibrated TWCCSN2 and the DTCCSN2-vs-m/z regression trendline for each class. Enforce the m/z constraints for class assignment (small molecules < m/z 550; lipids > m/z 300; peptide requires charge > 1), and if a feature falls outside the m/z constraints for its assigned class, select the next closest match by ΔCCS. Assign the class whose calibration yields the smallest ΔCCS, and record a calibration-effect score of 1–4 based on the percent difference to the next closest match (score 4 = ≥6%, score 3 = 3–6%, score 2 = 1–3%, score 1 = <1%; multiply charged assignments get default score 0).

## Related tools

- **MOCCal** (Python application that calibrates each arrival time with up to three class-specific power law equations, applies the m/z class-assignment rules, and selects the calibration with the smallest ΔCCS) — https://github.com/HinesLab/MOCCal
- **DEIMoS** (Extracts calibrant arrival time distributions from .mzML raw files and fits each to a Gaussian; the apex is recorded as the feature's arrival time (version 1.3.2 required for MOCCal RawDT mode)) — http://github.com/pnnl/deimos
- **Waters SYNAPT XS** (Traveling wave ion mobility mass spectrometer on which the TWIM-MS data to be class-assigned and calibrated are collected)

## Examples

```
python MOCCal_RawDT.py  # run in the DEIMoS 1.3.2 virtual environment with raw calibrant .mzML files and the experimental feature spreadsheet
```

## Evaluation signals

- On NIST SRM 1950 single-phase extract HILIC-IM-MS data, ≥88.6% of 44 lipid features and ≥93.3% of 15 small-molecule features are assigned to their structurally matched calibrant classes
- Per-class power law calibration RMSEs match ~0.28 Å² (lipid), ~0.58 Å² (small molecule), and ~0.12 Å² (peptide)
- Correct assignments for ≥70% of lipids and peptides with calibration errors within ±1% (best results at ±0.5%)
- Small molecules from the CCS Compendium are assigned to the small molecule calibrations 81–83% of the time with errors of ±0.5 to 5.0%
- Each feature carries a calibration-effect score consistent with the percent difference to the next closest match (1 = <1%, 2 = 1–3%, 3 = 3–6%, 4 = ≥6%)

## Limitations

- One small molecule (trazodone, m/z 372.159) was misassigned to the lipid calibration on NIST SRM 1950, showing cross-class misassignment can occur
- Single power law calibrations introduce calibration biases that may confound or prohibit use of calibrated TWCCSN2 values in feature identification
- MOCCal is currently limited to small molecule, lipid, and peptide calibrations within TWIM-MS constraints and is not yet extended to negative ionization mode data sets or other chemical classes
- TWIM platforms record arrival time at the detector, not drift time in the cell; the terms are used interchangeably in the software for convenience

## Evidence

- [results] m/z constraints for class assignment: [evidence span withheld — anchor status: not_found]
- [abstract] ΔCCS selection criterion: "uses the difference (in Å 2 ) between the calibrated TW CCS N2 value and DT CCS N2 vs m / z regression lines to determine the best calibration curve."
- [results] calibration-effect scoring: "A score of 4 indicates that there is a difference of 6% or greater between the calibrated CCS value derived from the assigned class and the next closest match."
- [results] Gaussian fit of arrival times via DEIMoS: [evidence span withheld — anchor status: not_found]
- [readme] RawDT mode requires DEIMoS: "If you plan to use raw calibration files, you will need to install pnnl’s DEIMoS, version 1.3.2 (http://github.com/pnnl/deimos)."
- [results] validated performance on complex mixture: "MOCCal provides the best assignment results for lipids and singly charged peptides when the calibration errors are ±0.5% but still achieves the correct assignment for ≥70% of lipids and peptides"
