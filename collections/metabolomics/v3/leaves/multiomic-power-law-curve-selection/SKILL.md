---
name: multiomic-power-law-curve-selection
description: Use when processing multiomic TWIM-MS data on Waters instruments where
  unknown features must be assigned to small molecule, lipid, or peptide calibrations
  and calibrated CCS values computed without prior identification.
when_to_use_negative:
- The data set is already a fully annotated feature table where features are identified
  and class assignment by ΔCCS is unnecessary
- Data were acquired in negative ionization mode or from chemical classes outside
  small molecule, lipid, and peptide scope (MOCCal does not yet calibrate these)
- The instrument is not a traveling wave ion mobility platform (e.g. drift tube IM-MS
  where DTCCSN2 is measured directly rather than calibrated)
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: MOCCal
  role: Python application that performs class-specific CCS calibration, biomolecular
    class assignment, and calibrated TWCCSN2 value selection
  repo: https://github.com/HinesLab/MOCCal
- name: DEIMoS
  role: Extracts arrival times from raw calibration .mzML files and fits them to a
    Gaussian distribution to provide the apex arrival time used in the power law regressions
  repo: http://github.com/pnnl/deimos
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1021/acs.analchem.3c04290
    title: 'MOCCal: A Multiomic CCS Calibrator for Traveling Wave Ion Mobility Mass
      Spectrometry'
  source_content_hash: sha256:1cf3ed85112e07a9adc532618a7a3461514da2e32673518b86ed443bbd1ce179
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
yes_rate: 1.0
n_claims: 8
---

# multiomic-power-law-curve-selection

## Summary

Automates selecting the class-specific power law calibration (small molecule, lipid, or peptide) that yields the most accurate TWCCSN2 value for each unknown feature in traveling wave ion mobility MS data. Uses ΔCCS-to-trendline selection without prior feature identification.

## When to use

Use when processing multiomic TWIM-MS data on Waters instruments where unknown features must be assigned to small molecule, lipid, or peptide calibrations and calibrated CCS values computed without prior identification.

## When NOT to use

- The data set is already a fully annotated feature table where features are identified and class assignment by ΔCCS is unnecessary
- Data were acquired in negative ionization mode or from chemical classes outside small molecule, lipid, and peptide scope (MOCCal does not yet calibrate these)
- The instrument is not a traveling wave ion mobility platform (e.g. drift tube IM-MS where DTCCSN2 is measured directly rather than calibrated)

## Inputs

- Experimental feature spreadsheet with feature identifiers, m/z, charge state, retention time, and arrival time values
- Raw calibration data as .mzML files (processed calibration data alternative: UserDT executable)
- Class-specific power law calibration parameters (from calibrant arrival time and reference drift tube CCS values)
- Reference drift tube CCS values from the Unified CCS Compendium

## Outputs

- Selected calibrated TWCCSN2 value per feature (smallest ΔCCS match)
- Biomolecular class assignment (small molecule, lipid, peptide) per feature
- Calibration effect scores (1–4 for singly charged; 0 default for multiply charged)
- Validation comparison of calibrated CCS values against reference drift tube values

## How to apply

Import experimental data as a spreadsheet with feature identifiers, m/z, charge state, retention time, and arrival time values, then calibrate every feature with up to three class-specific power law equations. Use the m/z constraints (small molecules less than m/z 550, lipids greater than m/z 300) and the smallest ΔCCS to the corresponding biomolecular class trendline to select the best calibrated TWCCSN2 value. Select the calibration that minimizes the difference in Å2 between the calibrated TWCCSN2 value and the DTCCSN2 vs m/z regression lines for the assigned biomolecular class, respecting that classes outside their m/z constraint fall back to the next closest match by ΔCCS. Multiply charged features (e.g. z=2 or 3) receive a default calibration effect score of 0; singly charged features get scores 1–4 based on percent difference between the assigned-class calibrated CCS and the next closest match. The rationale is that each biomolecular class has distinct fit parameters for the power law calibration curve, and structurally matched calibrations yield the lowest errors (e.g. lipid m/z 692.517: 0.18% error vs 1.8% and 2.6% with mismatched power laws).

## Related tools

- **MOCCal** (Python application that performs class-specific CCS calibration, biomolecular class assignment, and calibrated TWCCSN2 value selection) — https://github.com/HinesLab/MOCCal
- **DEIMoS** (Extracts arrival times from raw calibration .mzML files and fits them to a Gaussian distribution to provide the apex arrival time used in the power law regressions) — http://github.com/pnnl/deimos

## Examples

```
python MOCCal_RawDT.py --calibration_files calibration/*.mzML --experimental_data features.csv --output Output/
```

## Evaluation signals

- Calibrated TWCCSN2 values for multiply charged polyalanine peptides are within 0.62% of drift tube reference values
- Poly-Ala6 and Poly-Ala7 calibrated values fall within 0.6% and 1.2% of reference, respectively
- ADH/enolase digest peptide calibrated TWCCSN2 values are within 2.6% of reference values
- Lipid feature at m/z 692.517 calibrated with lipid-fit power law yields 0.18% error vs 1.8% and 2.6% with small molecule/peptide fits
- Calibration effect scores for singly charged features fall in the 1–4 range with score 1 indicating <1% difference between assigned-class and next closest calibration

## Limitations

- MOCCal is currently limited to small molecule, lipid, and peptide calibrations within TWIM-MS platforms
- MOCCal is not yet extended to negative ionization mode data sets
- Other endogenous or xenobiotic chemical classes beyond small molecule, lipid, and peptide are not yet supported
- User-driven expansion is welcomed but not implemented in the current release

## Evidence

- [results] MOCCal calibrates every experimental arrival time with up to three class-specific calibration curves: "MOCCal calibrates every experimental arrival time with up to three class-specific calibration curves"
- [results] less than m/z 550 for small molecules; greater than m/z 300 for lipids: [evidence span withheld — anchor status: not_found]
- [results] The calibrated TWCCSN2 value with the smallest ΔCCS to the corresponding trendline is selected as the correct match: "The calibrated TWCCSN2 value with the smallest ΔCCS to the corresponding trendline is selected as the correct match"
- [results] MOCCal will calculate up to three calibrated TWCCSN2 values using each set of class-specific calibration parameters: "MOCCal will calculate up to three calibrated TWCCSN2 values using each set of class-specific calibration parameters"
- [readme] MOCCal offers class assignment and CCS calculations without need for identifying the features first: "MOCCal offers class assignment and CCS calculations without need for identifying the features first."
- [results] MOCCal provides the best assignment results for lipids and singly charged peptides when the calibration errors are ±0.5%: "MOCCal provides the best assignment results for lipids and singly charged peptides when the calibration errors are ±0.5% but still achieves the correct assignment for ≥70% of lipids and peptides with"
