---
name: small-molecule-ccs-verification-against-compendium
description: Use when a TWIM-MS feature list contains small-molecule features (singly
  charged, m/z < 550) whose calibrated TWCCSN2 values need validation against drift-tube
  reference CCS values before feature identification.
when_to_use_negative:
- The feature is multiply charged (charge > 1), in which case it is treated as a peptide
  and receives default calibration-effect score 0 rather than the small-molecule assignment
  path
- The data were acquired in negative ionization mode, since MOCCal has not yet been
  extended to negative ionization mode data sets
- Features have already been structurally identified and matched to calibrants, making
  automated class assignment unnecessary
edam_operation: http://edamontology.org/operation_0326
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: MOCCal
  role: Python application that calibrates each feature's arrival time with up to
    three class-specific power law equations, selects the best calibration by smallest
    ΔCCS, and assigns biomolecular class without prior identification
  repo: https://github.com/HinesLab/MOCCal
- name: DEIMoS
  role: Used to extract arrival-time distributions from raw .mzML calibration files
    and fit each to a Gaussian, recording the apex as the calibrant's arrival time
  repo: http://github.com/pnnl/deimos
- name: CCSBase
  role: Reference resource of CCS values consulted alongside the Unified CCS Compendium
    for comparison against calibrated TWCCSN2 values
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/acs.analchem.3c04290
    title: 'MOCCal: A Multiomic CCS Calibrator for Traveling Wave Ion Mobility Mass
      Spectrometry'
  source_content_hash: sha256:d1e30fc415f6ca9dab219cde8788876427f9256e61a081c0b446fe389347f993
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/HinesLab/MOCCal
  tools:
  - CCS Compendium
  - CCSBase
  - DEIMoS
  - MOCCal
  - Waters AQUITY UPLC BEH amide
  - Waters SYNAPT XS
  edam_operation: http://edamontology.org/operation_0326
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8947368421052632
n_claims: 19
---

# small-molecule-ccs-verification-against-compendium

## Summary

Verify calibrated traveling-wave CCS values for small-molecule features (m/z < 550) by comparing them against DTCCSN2 reference values from the Unified CCS Compendium and LipidPioneer databases, using MOCCal's class assignment and ΔCCS selection to confirm assignment accuracy.

## When to use

Use when a TWIM-MS feature list contains small-molecule features (singly charged, m/z < 550) whose calibrated TWCCSN2 values need validation against drift-tube reference CCS values before feature identification.

## When NOT to use

- The feature is multiply charged (charge > 1), in which case it is treated as a peptide and receives default calibration-effect score 0 rather than the small-molecule assignment path
- The data were acquired in negative ionization mode, since MOCCal has not yet been extended to negative ionization mode data sets
- Features have already been structurally identified and matched to calibrants, making automated class assignment unnecessary

## Inputs

- Spreadsheet of experimental features with feature identifiers, m/z, charge state, retention time, and arrival time values
- Class-specific power law calibration parameters derived from Waters Major Mix IMS/Tof Calibration Kit and SPLASH Lipidomix calibrant arrival times (Gaussian apex arrival times extracted with DEIMoS 1.3.2)
- Reference DTCCSN2 values from the Unified CCS Compendium (Supporting Information Document 2, Tables 1–3)
- Reference exact masses and CCS values from LipidPioneer and CCS Compendium databases

## Outputs

- Per-feature assigned biomolecular class (small molecule, lipid, or peptide)
- Calibrated TWCCSN2 value per feature from the smallest-ΔCCS class-specific power law
- Calibration-effect score (1–4, or 0 for multiply charged assignments) per feature
- Per-feature error (%) versus reference CCS values from LipidPioneer/CCS Compendium

## How to apply

Import the experimental feature table (feature identifier, m/z, charge state, retention time, arrival time) into MOCCal, which calibrates each singly charged feature with up to three class-specific power law equations built from calibrants like the Waters Major Mix IMS/Tof Calibration Kit and SPLASH Lipidomix. Apply the class-assignment rule that small molecules are m/z < 550, and if a feature falls outside the assigned class's m/z constraints, MOCCal falls back to the next closest match by ΔCCS. Select the calibration whose calibrated TWCCSN2 value has the smallest ΔCCS to the corresponding DTCCSN2 vs m/z regression trendline derived from the Unified CCS Compendium. Assign a calibration-effect score of 1–4 based on the percent difference to the next closest match (score 1 < 1%, score 2 = 1–3%, score 3 = 3–6%, score 4 ≥ 6%). Evaluate exact masses and CCS values against LipidPioneer and CCS Compendium databases, and flag cross-class misassignments such as the trazodone feature (m/z 372.159) that was assigned to the lipid calibration.

## Related tools

- **MOCCal** (Python application that calibrates each feature's arrival time with up to three class-specific power law equations, selects the best calibration by smallest ΔCCS, and assigns biomolecular class without prior identification) — https://github.com/HinesLab/MOCCal
- **DEIMoS** (Used to extract arrival-time distributions from raw .mzML calibration files and fit each to a Gaussian, recording the apex as the calibrant's arrival time) — http://github.com/pnnl/deimos
- **CCSBase** (Reference resource of CCS values consulted alongside the Unified CCS Compendium for comparison against calibrated TWCCSN2 values)

## Examples

```
python MOCCal_RawDT.py  # run in the DEIMoS 1.3.2 virtual environment after installing pnnl's DEIMoS (http://github.com/pnnl/deimos)
```

## Evaluation signals

- On NIST SRM 1950 single-phase extract data, ≥ 93.3% of the 15 small-molecule features are assigned to their structurally matched calibrants
- Per-class RMSE of the calibration regressions matches ~0.58 Å2 for small molecules (vs ~0.28 Å2 for lipids and ~0.12 Å2 for peptides)
- Calibration-effect scores are populated for each feature (1–4 for singly charged, 0 for multiply charged), with scores of 1–2 (<3% difference to next closest match) indicating confident assignments
- Small-molecule features from the CCS Compendium are assigned to the small molecule calibrations 81–83% of the time with errors of ±0.5 to 5.0%
- Cross-class misassignments are flagged and quantified, e.g. the trazodone feature (m/z 372.159) assigned via the lipid calibration shows ~3.2% error

## Limitations

- MOCCal is currently limited to small molecule, lipid, and peptide calibrations within TWIM-MS constraints
- MOCCal has not yet been extended to negative ionization mode data sets or other endogenous/xenobiotic chemical classes
- Calibration biases introduced by applying a single calibration equation may confound feature identification, motivating the multi-class approach

## Evidence

- [results] Class assignment rule with m/z constraints and fallback: [evidence span withheld — anchor status: not_found]
- [abstract] Selection criterion based on smallest ΔCCS: "uses the difference (in Å 2 ) between the calibrated TW CCS N2 value and DT CCS N2 vs m / z regression lines to determine the best calibration curve."
- [results] Calibration-effect score thresholds: "A score of 4 indicates that there is a difference of 6% or greater between the calibrated CCS value derived from the assigned class and the next closest match. A score of 3 corresponds to a"
- [results] Validation against LipidPioneer and CCS Compendium: "evaluated the exact masses and CCS values of the features against the LipidPioneer and CCS Compendium databases"
- [readme] README usage instructions for raw calibration data path: "If you plan to use raw calibration files, you will need to install pnnl’s DEIMoS, version 1.3.2 (http://github.com/pnnl/deimos). After DEIMoS is set up, you can then run MOCCal_RawDT.py"
