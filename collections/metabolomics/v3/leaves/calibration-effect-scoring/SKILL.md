---
name: calibration-effect-scoring
description: Use when assigning unknown TWIM-MS features to small-molecule, lipid,
  or peptide CCS calibrations and you need to flag which class assignments are well-supported
  (score 1-2) vs ambiguous cross-class matches (score 3-4).
when_to_use_negative:
- The features are multiply charged — MOCCal assigns them a default calibration-effect
  score of 0, so the score carries no discrimination information
- Data are not from a traveling-wave IM-MS platform or fall outside the small molecule,
  lipid, and peptide calibration classes MOCCal was built around (e.g., negative ionization
  mode datasets or other endogenous/xenobiotic classes, which are not yet supported)
- A single class-specific calibration is already known to apply — the score is designed
  to compare across up to three competing class calibrations
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_0121
tools:
- name: MOCCal
  role: Computes up to three class-specific calibrated TWCCSN2 values per feature,
    selects the smallest-ΔCCS calibration, and assigns the 1-4 calibration-effect
    score
  repo: https://github.com/HinesLab/MOCCal
- name: DEIMoS
  role: Extracts arrival-time distributions from raw calibrant .mzML files and fits
    Gaussians to obtain the apex arrival times used to build the class-specific power
    law calibrations underlying the scores
  repo: http://github.com/pnnl/deimos
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/acs.analchem.3c04290
    title: 'MOCCal: A Multiomic CCS Calibrator for Traveling Wave Ion Mobility Mass
      Spectrometry'
  source_content_hash: sha256:d1de1b9177ac3859bba7ffe587d6ea2bd2b9dccef55459b89d205270f8e13f97
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 13
---

# calibration-effect-scoring

## Summary

A MOCCal scoring scheme that quantifies how discriminating a biomolecular class assignment is for TWIM-MS features: after calibrating each feature with up to three class-specific power law equations, each assignment receives a 1-4 calibration-effect score based on the percent difference between the calibrated TWCCSN2 value from the assigned class and the next closest match.

## When to use

Use when assigning unknown TWIM-MS features to small-molecule, lipid, or peptide CCS calibrations and you need to flag which class assignments are well-supported (score 1-2) vs ambiguous cross-class matches (score 3-4).

## When NOT to use

- The features are multiply charged — MOCCal assigns them a default calibration-effect score of 0, so the score carries no discrimination information
- Data are not from a traveling-wave IM-MS platform or fall outside the small molecule, lipid, and peptide calibration classes MOCCal was built around (e.g., negative ionization mode datasets or other endogenous/xenobiotic classes, which are not yet supported)
- A single class-specific calibration is already known to apply — the score is designed to compare across up to three competing class calibrations

## Inputs

- Feature spreadsheet with feature identifiers, m/z, charge state, retention time, and arrival time values
- Class-specific power law calibration parameters derived from calibrant .mzML files (Waters Major Mix IMS/Tof Calibration Kit, SPLASH Lipidomix) processed in DEIMoS 1.3.2
- DTCCSN2-vs-m/z regression trendlines from Unified CCS Compendium reference values
- Reference CCS databases for validation (LipidPioneer, CCS Compendium)

## Outputs

- Per-feature table of assigned biomolecular class, calibrated TWCCSN2 value, and calibration-effect score (1-4, or 0 for multiply charged assignments)
- ΔCCS values (Å2) for each candidate class calibration per feature
- Error metrics vs. LipidPioneer/CCS Compendium reference values (e.g., percent error per feature)

## How to apply

Calibrate each singly charged feature with up to three class-specific power law equations (small molecule, lipid, peptide) built from calibrant arrival times (Gaussian-fitted apexes from DEIMoS on .mzML files) and reference DTCCSN2 values from the Unified CCS Compendium. For each candidate calibration, compute ΔCCS as the difference (in Å2) between the calibrated TWCCSN2 value and the DTCCSN2-vs-m/z regression trendline, then select the calibration with the smallest ΔCCS, applying the m/z constraints (small molecules < m/z 550; lipids > m/z 300) with fallback to the next closest match when constraints are violated. Score the assignment 1-4 by the percent difference between the calibrated CCS from the assigned class and the next closest class match: score 1 = <1%, score 2 = 1-3%, score 3 = 3-6%, score 4 = ≥6%; multiply charged assignments receive a default score of 0. High scores indicate the class assignment (and calibrated CCS) is not structurally discriminated — e.g., the trazodone feature at m/z 372.159 in NIST SRM 1950 was assigned via the lipid calibration despite being a small molecule, with ~3.2% error.

## Related tools

- **MOCCal** (Computes up to three class-specific calibrated TWCCSN2 values per feature, selects the smallest-ΔCCS calibration, and assigns the 1-4 calibration-effect score) — https://github.com/HinesLab/MOCCal
- **DEIMoS** (Extracts arrival-time distributions from raw calibrant .mzML files and fits Gaussians to obtain the apex arrival times used to build the class-specific power law calibrations underlying the scores) — http://github.com/pnnl/deimos

## Examples

```
python MOCCal_RawDT.py  # run in DEIMoS 1.3.2 virtual environment with calibrant .mzML files and feature template in the RawData folder
```

## Evaluation signals

- Score thresholds are honored: score 1 assignments differ <1% between assigned class and next closest match, score 2 = 1-3%, score 3 = 3-6%, score 4 = ≥6%
- Multiply charged features receive the default score of 0, not a 1-4 score
- On NIST SRM 1950 single-phase extract, ≥88.6% of 44 lipid features and ≥93.3% of 15 small-molecule features are assigned to their structurally matched calibrants
- Per-class RMSE of calibrated values match ~0.28 Å2 (lipid), ~0.58 Å2 (small molecule), ~0.12 Å2 (peptide)
- Cross-class misassignments are recoverable from scores: the trazodone feature (m/z 372.159) assigned via lipid calibration shows ~3.2% error, and peptide-calibrated lysophospholipids fall in the 0.5-1.8% error range

## Limitations

- MOCCal is scoped to small molecule, lipid, and peptide calibrations on TWIM-MS platforms; applying a single calibration equation instead can introduce calibration biases that confound feature identification
- MOCCal has not yet been extended to negative ionization mode datasets or other endogenous or xenobiotic chemical classes
- Multiply charged assignments default to score 0, so the calibration-effect score does not assess their assignment confidence
- If a feature falls outside the m/z constraints of its assigned class (small molecules < m/z 550, lipids > m/z 300), the next closest match by ΔCCS is selected, which can produce cross-class assignments like the trazodone example

## Evidence

- [results] Score thresholds and meaning: "A score of 4 indicates that there is a difference of 6% or greater between the calibrated CCS value derived from the assigned class and the next closest match."
- [abstract] ΔCCS selection criterion: "uses the difference (in Å 2 ) between the calibrated TW CCS N2 value and DT CCS N2 vs m / z regression lines to determine the best calibration curve."
- [results] Multi-calibration computation per feature: "MOCCal will calculate up to three calibrated TWCCSN2 values using each set of class-specific calibration parameters"
- [readme] Tool scope from README: "Functionality includes collision cross section (CCS) calibration, experimental data biomolecular class assignment, and experimental class-specific CCS calculations."
